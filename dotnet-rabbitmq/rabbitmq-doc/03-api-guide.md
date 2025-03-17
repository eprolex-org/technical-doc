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
    AutomaticRecoveryEnabled = true,
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



## La `Queue`

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

