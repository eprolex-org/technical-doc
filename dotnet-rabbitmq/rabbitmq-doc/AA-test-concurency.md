# AA Test de concurrence

## Le Producer

```cs
// On fixe le nombre de message : identique chez le worker
const int nbOfMessages = 30;

var factory = new ConnectionFactory { HostName = "localhost" };
await using var connection = await factory.CreateConnectionAsync();
await using var channel = await connection.CreateChannelAsync();

await channel.QueueDeclareAsync(
    queue: "test-concurency",
    durable: true,
    autoDelete: false,
    exclusive: false,
    arguments: null
);

var body = "Hello World!"u8.ToArray();

// On envoie le nombre prédéfini de messages
for (var i = 0; i < nbOfMessages; i++) {
    
    await channel.BasicPublishAsync(
        exchange: string.Empty,
        routingKey: "test-concurency",
        body: body
    );
    
}
```



## Le `Worker`

```cs

var processorCount = Environment.ProcessorCount;
var k = 3;

Console.WriteLine($"Nombre de Thread: {processorCount}");

var consumerDispatchConcurrency = processorCount;
var prefetchCount = processorCount * k;
const int maxNumberOfMessages = 30;

var countMessages = 0;
var firstMessageConsumed = false;
var sw = new Stopwatch();

var factory = new ConnectionFactory { HostName = "localhost" , ConsumerDispatchConcurrency = (ushort)consumerDispatchConcurrency };
await using var connection = await factory.CreateConnectionAsync();
await using var channel = await connection.CreateChannelAsync();

await channel.QueueDeclareAsync(
    queue: "test-concurency",
    durable: true,
    autoDelete: false,
    exclusive: false,
    arguments: null
);

await channel.BasicQosAsync(prefetchCount: (ushort)prefetchCount, prefetchSize: 0, global: false);

var consumer = new AsyncEventingBasicConsumer(channel);

consumer.ReceivedAsync += async (sender, ea) => {
    
    if (!firstMessageConsumed) {
        sw.Start();
        firstMessageConsumed = true;
    }
    
    var currentCount = Interlocked.Increment(ref countMessages);
    
    var body = ea.Body.ToArray();

    var message = Encoding.UTF8.GetString(body);

    Thread.Sleep(TimeSpan.FromSeconds(1));
    
    Console.WriteLine($"receive {message} {currentCount} [X]");
    
    if (currentCount >= maxNumberOfMessages) {
        sw.Stop();
        
        Console.WriteLine($"\n[x] Nombre de messages traités : {currentCount}");
        Console.WriteLine($"[x] Temps total : {sw.Elapsed.Seconds} secondes {sw.Elapsed.Milliseconds} ms");
        Console.WriteLine($"[x] Débit : un message est traité en {sw.Elapsed.TotalSeconds / currentCount} sec");

        // Arrêt de l'application
        Environment.Exit(0);
    }
    
    await channel.BasicAckAsync(ea.DeliveryTag, false);
};

await channel.BasicConsumeAsync(
    queue: "test-concurency",
    autoAck: false,
    consumer: consumer
);

Console.ReadKey();
```

Le plus efficae est d'utiliser le nombre de processeur avec `Environment.ProcessorCount` comme valeur de `ConsumerDispatchConcurency` et de multiplié cette valeur par un coefficient `k` plutôt petit (par exemple `3`) pour `PrefetchCount`. Ainsi, le `Worker` est sûr d'voir un ou deux message en attente.

On utilise `Interlocked.Increment(ref countMessages)` pour avoir un compteur fiable entre plusieurs `Thread`.



### `prefetchCount`

Ce renseigne dans les `BasicQos` , tout de suite après la déclaration de la `queue` :

```cs
await channel.QueueDeclareAsync(
    queue: "test-concurency",
    durable: true,
    autoDelete: false,
    exclusive: false,
    arguments: null
);

await channel.BasicQosAsync(prefetchCount: 36, prefetchSize: 0, global: false);
```



### `ConsumerDispatchConcurency`

Se renseigne dans la `ConnectionFactory` :

```cs
var factory = new ConnectionFactory { 
    HostName = "localhost" , 
    ConsumerDispatchConcurrency = 12 // c'est de type ushort
};
```

