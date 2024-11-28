# 04 `.net` et `Rabbitmq`



## Installation

```bash
dotnet add package RabbitMQ.client
```



## Les classes de base

### `ConnectionFactory`

Permet de construire un objet `IConnection`.

### `IConnection`

Reprèsente une connexion à `rabbitmq` utilisant `AMQP`.

### `IModel`

Reprèsente un canal (`channel`) vers le serveur `AMQP` au travers duquel on peut réaliser des `actions` : `créer une queue`, `envoyer un message`, ...

### `QueueingBasicConsumer`

C'est le moyen le plus commun pour recevoir des `messages`.

### `protocols`

Permet de choisir quelle version d'`AMQP` on veut utiliser.



## Créer une `connection`

```cs
var factory = new ConnectionFactory() { HostName = "localhost" };

// en précisant les UserName et Password
var factory = new ConnectionFactory
{
    HostName = "localhost", 
    UserName = "my-username", 
    Password = "my-p@ssw0rd"
};

using var connection = factory.CreateConnection();
```

> Par défaut on a les crédentials suivant :
>
> ```cs
> // Default password (value: "guest").
> public const string DefaultPass = "guest";
> 
> // Default user name (value: "guest").
> public const string DefaultUser = "guest";
> ```



## Créer un `channel`

```cs
using var channel = connection.CreateModel();
```

Méthodes courantes de `channel` :

- `BasicPublish` pour envoyer un message
- `ExchangeDeclare`
- `QueueBind` lie une `queue` à un `exchange`
- `QueueDeclare`



## Créer un `Consumer`

```cs
var consumer = new EventingBasicConsumer(channel);
```



### Consommer un message : `BasicConsume`

```cs
consumer.Received += (model, ea) =>
{
    var body = ea.Body.ToArray();
    var message = Encoding.UTF8.GetString(body);

    Console.WriteLine($"Message received: {message}");
};
```

`ea` pour `EventArgs`.

```cs
channel.BasicConsume(
    queue: "m1-s3",
    consumer: consumer,
    autoAck: true
);
```



