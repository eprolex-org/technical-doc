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



### Encoder un `message` en `Bytes`

```cs
const string message = "this is my first message";

var bytesMessage = Encoding.UTF8.GetBytes(message);
```



### Publier un `message` : `BasucPublish`

```cs
channel.BasicPublish(
    exchange: "",
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
consumer.Received += (model, eventArgs) =>
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

`global` : le réglage est-il pour tous les `Consumers`
