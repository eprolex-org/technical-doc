# 05 `RabbitMQ` Client Library

C'est une implémentation en `c#` de `AMQP` `0.9.1` avec quelques `abstractions` en plus.



## `Implémentation`

### `IModel`

Représente un `channel` `AMQP` et fournit les opérations `AMQP`.



### `IConnection`

Représente une `connexion` avec `RabbitMQ`.



### `ConnectionFactory`

Construit une instance de `IConnection`.



### `ConnectionParameter`

Configure le `ConnectionFactory`.



### `QueuingBasicConsumer`

Reçoit les `messages` délivrés par le serveur.



## Connection à un `Message Broker`

```cs
ConnectionFactory factory = new ConnectionFactory {
    HostName = "localhost",
    UserName = "guest",
    Password = "guess"
};

IConnection connection = factory.CreateConnection();
```



## Créer un `channel`

```cs
IModel _model = connection.CreateModel();
```



## Créer des `Exchanges` et des `Queues`

```cs
channel.ExchangeDeclare("MyExchange", "direct");

channel.QueueDeclare("MyQueue");

channel.QueueBind("MyQueue", ExchangeName, "");
```



