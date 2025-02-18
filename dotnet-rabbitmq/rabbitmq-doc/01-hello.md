# 01 `Hello World`

## Le `producer` : `send`

```cs
var factory = new ConnectionFactory { HostName = "localhost" };

await using var connection = await factory.CreateConnectionAsync();

await using var channel = await connection.CreateChannelAsync();

await channel.QueueDeclareAsync("hello_world", exclusive: false);

while (Console.ReadKey().Key != ConsoleKey.Q) {
    
    var messageEncoded = Encoding.UTF8.GetBytes(messsage());
    
    await channel.BasicPublishAsync("", "hello_world", false, messageEncoded);
}

string messsage() => $"Hello receiver {DateTime.Now.TimeOfDay}!";

```

- on instancie une `factory` avec `new ConnectionFactory`
- On créé une `connection` (`TCP`) : `CreateConnectionAsync`
- On créé un `channel` : `CreateChannelAsync`
- On déclare une `queue` : `QueueDeclareAsync`
- On transforme notre `message` en tableau de `bytes` : `Encoding.UTF8.GetBytes`
- On envoie notre `message` : `BasicPublishAsync`



## Le `consumer` : `reveive`

```cs
var factory = new ConnectionFactory { HostName = "localhost" };

await using var connection = await factory.CreateConnectionAsync();
await using var channel = await connection.CreateChannelAsync();

await channel.QueueDeclareAsync("hello_world", exclusive: false);

var consumer = new AsyncEventingBasicConsumer(channel);

consumer.ReceivedAsync += (sender, ea) => {
    var body = ea.Body;
    
    var message = Encoding.UTF8.GetString(body.ToArray());
    Console.WriteLine($" [x] Received {message}");

    return Task.CompletedTask;
};

await channel.BasicConsumeAsync("hello_world", true, consumer);

Console.ReadKey();
```

- On créé un `consumer` : `new AsyncEventingBasicConsumer`
- On l'abonne à l'événement : `ReceivedAsync`
- On transforme le tableau de `bytes` en message lisible : `Encoding.UTF8.GetString`
- On consomme les messages (on s'abonne) : `BasicConsumeAsync`

