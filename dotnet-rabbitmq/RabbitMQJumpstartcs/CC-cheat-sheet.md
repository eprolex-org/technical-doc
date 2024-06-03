# CC - Copion `RabbitMQ`

## Exercice 1

### Créer une `connection` :

```cs
var factory = new ConnectionFactory
{
    HostName = "localhost",
    UserName = "guest",
    Password = "guest",
};

using var connection = factory.CreateConnection();
```

ou

```cs
var factory = new ConnectionFactory();

using var connection = factory.CreateConnection("localhost");
```



### Crée un `channel`

```cs
using var channel = connection.CreateModel();
```



### déclarer (créer si elle n'existe pas) une `Queue`

```cs
channel.QueueDeclare(
    queue: "firstexample",
    durable: false,
    exclusive: false,
    autoDelete: true
);
```

`exclusive: true` la `queue` ne peut être utilisé que par l'application ayant créé la connexion, si la connexion se termine, la `queue` est détruite.

### Encoder un `message` en `Bytes`

```cs
const string message = "this is my first message";

var bytesMessage = Encoding.UTF8.GetBytes(message);
```



### Publier un `message` : `BasicPublish`

```cs
channel.BasicPublish(
    exchange: string.Empty,
    routingKey: "firstexample",
    basicProperties: null,
    body: bytesMessage
);
```



### Créer un `consumer`

```cs
var consumer = new EventingBasicConsumer(channel);
```



### Gérer la reception d'un `message`

```cs
consumer.Received += (_, eventArgs) =>
{
    var body = eventArgs.Body.ToArray();
    var message = Encoding.UTF8.GetString(body);
    Console.WriteLine($"Received message: {message}");
};
```



### Consommer les `messages` : `BasicConsume`

```cs
channel.BasicConsume(
    queue: "firstexample",
    autoAck: true,
    consumer: consumer
);
```



### Accusé de reception manuel (exo 3)

```cs
channel.BasicConsume(
    queue: "secondexample",
    consumer: consumer,
    autoAck: false // <- ici
);
```

```cs
channel.BasicAck(deliveryTag: eventArgs.DeliveryTag, multiple: false);
```



### Réglage de la répartition des `messages` : `channel.BasicQos` (exo 4)

```cs
channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
```

`prefetchSize` : taille max message 0 = pas de limite

`prefetchCount` : nombre de `messages` reçu

`global` : le nombre de `message` est-il pour tous les `Consumers`

> Si `prefetchCount` est de `12` et `global` est à `true`, c'est `12 messages` pour tous les `consumers`, Si il y a `3 consumers`, cela fait `4 messages` par `Consumers`. Par contre si `global` est à `false`, cela ferait `12 messages` par `consumer`, donc `36` au total.



### Créer un `Exchange` : `ExchangeDeclare`

```cs
channel.ExchangeDeclare(
    exchange: "app_exchange",
    ExchangeType.Direct
);
```



### Lier une `Queue` à un `Exchange` : `QueueBind`

```cs
channel.QueueBind(
    queue: "customer_queue",
    exchange: "app_exchange",
    routingKey: "order"
);
```



## Concurrence des `Consumers`

 ### Nombre de `Consumer` concurrent : `ConsumerDispatchConcurrency`

```cs
var coresNumber = Environment.ProcessorCount;

factory.ConsumerDispatchConcurrency = coresNumber;

using var connection = factory.CreateConnection("localhost");
```



### `async` consumer

Si le `consumer` exécute une lambda asynchrone :

```cs
factory.DispatchConsumersAsync = true;

// ...

var consumer = new AsyncEventingBasicConsumer(channel);

consumer.Received += async (_, ea) =>
{
    var body = ea.Body.ToArray();
    var message = Encoding.UTF8.GetString(body);

    await SimulerTraitementLong();

    channel.BasicAck(ea.DeliveryTag, false);
};
```

