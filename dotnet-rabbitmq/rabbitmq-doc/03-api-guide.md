# 03 Guide de l'`api`

## `IConnection`

```cs
var factory = new ConnectionFactory
{
    HostName = "localhost",
    UserName = "guest", // valeur par défaut
    Password = "guest", // valeur par défaut
    Port = 5672, // valeur par défaut
    AutomaticRecoveryEnabled = true,
    ConsumerDispatchConcurrency = concurentConsumers
};

await using var connection = await factory.CreateConnectionAsync();
```

`ConsumerDispatchConcurrency` nombre de `consumers` concurrents, le nombre idéal étant `Environment.ProcessorCount`.



### gestion de la déconnexion `AutomaticRecoveryEnabled`

```cs
var factory = new ConnectionFactory
{
    AutomaticRecoveryEnabled = true, // par défaut
    // ...
};
```

```cs
await using var connection = await factory.CreateConnectionAsync();

connection.ConnectionShutdownAsync += (sender, @event) => {
    Console.WriteLine("connection killed");

    return Task.CompletedTask;
};

connection.RecoverySucceededAsync += (sender, @event) => {
    Console.WriteLine("connection recovery");

    return Task.CompletedTask;
};
```

```bash
connection killed # brew services stop rabbitmq
connection recovery # brew services start rabbitmq
```



## déconnexion. de `RabbitMQ`

```cs
await channel.CloseAsync();
await connection.CloseAsync();

await channel.DisposeAsync();
await connection.DisposeAsync();
```

Ou utiliser `await using` :

```cs
await using var connection = await factory.CreateConnectionAsync();
```



## Création d'un `channel`

```cs
await using var channel = await connection.CreateChannelAsync();
```



### gestion des erreurs dans le `Channel`

```cs
channel.CallbackExceptionAsync += (sender, @event) => {
    Console.WriteLine("An exception was happened");

    Console.WriteLine($"message dans CallbackExceptionAsync {@event.Exception.Message}");

    return Task.CompletedTask;
};
```

Remonte aussi les `exception` du `consumer`. On peut centraliser les `log`. Ici.



## La `Queue` : `QueueDeclareAsync`


```cs
await channel.QueueDeclareAsync(
    queue: "concurrency_queue", // nom de la queue
    durable: false,
    autoDelete: true,
    exclusive: false
);
```

Vérifie que la `queue` existe,  en créé une si nécessaire.



### Information en retour

```cs
var response = await channel.QueueDeclareAsync(
    queue: "concurrency_queue",
    durable: false,
    autoDelete: true,
    exclusive: false
);

Console.WriteLine($"Nombre de consumer(s) {response.ConsumerCount}");
Console.WriteLine($"Nombre de message(s) {response.MessageCount}");
```

```
Nombre de consumer(s) 2
Nombre de message(s) 216
```



### Paramètres de `queue`
```cs
Task<QueueDeclareOk> QueueDeclareAsync(
    string queue = "",
    bool durable = false,
    bool exclusive = false,
    bool autoDelete = false,
    IDictionary<string, object>? arguments = null,
    CancellationToken cancellationToken = default
);
```

`queue` : nom de la `queue`.

`durable` : persiste à un redémarage de `RabbitMQ`

`exclusive` : exclusive à une seule `connexion`

`autoDelete` : est automatiquement supprimée s'il n'y a plus de `consumer`



## `Quality Of Service` : `BasicQosAsync`

Définit comment les `messages` sont délivrés ax `consumers`


```cs
await _channel.BasicQosAsync(
    prefetchSize: 0,
    prefetchCount: processorCount,
    global: false
);
```



### Arguments
```cs
Task BasicQosAsync(
    uint prefetchSize,
    ushort prefetchCount,
    bool global,
    CancellationToken cancellationToken = default
);
```

`prefetchSize` taille en octets maximale d'un `message`, `0` pas de limite.

`prefetchCount` nombre maximale de `messages` non acquittés qu'un `consumer` peut recevoir avant d'envoyer un `ACK` .

`global` s'applique seulement au `consumer` actuel ou à tous.




## Le `consumer`

```cs
var consumer = new AsyncEventingBasicConsumer(channel);
```

```cs
consumer.ReceivedAsync += async (sender, ea) => { ... };
```



### Récupérer le `payload` du `message`

```cs
var body = ea.Body.ToArray();
```



### récupérer le `channel` dans la lambda

```cs
var innerChannel = ((AsyncEventingBasicConsumer)sender).Channel;
```

Évite l'utilisation de `channel` simultanement sur plusieurs `Thread` :

> [!WARNING]
>
> Captured variable is disposed in the outer scope



### S'abonner aux `messages` : `BasicConsumeAsync`

```cs
await channel.BasicConsumeAsync(
    queue: "concurrency_queue",
    autoAck: false,
    consumer: consumer
);
```



## Accusé de reception (`ACK`) : `BasicAckAsync`

```cs
Task BasicAckAsync(
    ulong deliveryTag,
    bool multiple,
    CancellationToken cancellationToken = default
);
```

`deliveryTag` : Identifiant unique du `message`

`multiple` : Si `true` acquites aussi tous les messages précédents.

> [!WARNING]
>
> `!` si plusieurs `consumers` , `multiple = true` entraine des `acks` involontaires `!`.

```cs
await Channel.BasicAckAsync(
    deliveryTag: ea.DeliveryTag,
    multiple: false
);
```



