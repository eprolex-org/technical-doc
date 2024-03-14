# `Resquest/Response` pattern

On utilise les termes `Client`/`Server` plutôt que `Publisher`/`Consumer`



## Client

```cs
var factory = new ConnectionFactory() { HostName = "localhost" };

using var connection = factory.CreateConnection();
using var channel = connection.CreateModel();

channel.QueueDeclare(
    queue: "letterBox", 
    durable: false, 
    exclusive: false,
    autoDelete: false,
    arguments: null
);

var consumer = new EventingBasicConsumer(channel);

consumer.Received += (model, ea) =>
{
    var body = ea.Body
};
```





## Server

```cs
```

