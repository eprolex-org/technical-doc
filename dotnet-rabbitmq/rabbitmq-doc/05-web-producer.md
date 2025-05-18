# 05 Implémentation d'un `Producer` dans une `WebApp`

## `ProducerFactory`

```cs
public static class ProducerFactory
{
    public static async Task<Producer> CreateProducerAsync() {
        
        var factory = new ConnectionFactory { HostName = "localhost" };
        
        var connection = await factory.CreateConnectionAsync();
        var channel = await connection.CreateChannelAsync();
        
        await channel.QueueDeclareAsync(
            queue: "web-queue",
            durable: true,
            exclusive: false,
            autoDelete: false
        );

        return new Producer(connection, channel);
    }
}
```



## `Producer`

Expose seulement `PublishMessageAsync` et `DisposeAsync`.

```cs
public class Producer(IConnection connection, IChannel channel) : IAsyncDisposable
{
    public async Task PublishMessageAsync(string message) {
        
        var body = Encoding.UTF8.GetBytes(message);

        await channel.BasicPublishAsync(
            exchange: string.Empty,
            routingKey: "web-queue",
            body: body
        );
    }

    public async ValueTask DisposeAsync() {
        Console.WriteLine("I'am disposing async correctly ...");
        
        await connection.CloseAsync();
        await connection.DisposeAsync();

        await channel.CloseAsync();
        await channel.DisposeAsync();
        
        Console.WriteLine("... I'am disposed");
    }

}
```

> La `connection` et le `channel` sont passés à la classe `Producer` et c'est celle-ci qui les `dispose` proprement.
>
> La classe `ProducerFactory` peut donc être `static`.

## `Program.cs`

```cs
// ...

var p = await ProducerFactory.CreateProducerAsync();
builder.Services.AddSingleton(_ => p);

// ...

app.MapGet(
    "/producer/{message}", 
    async ([FromServices] Producer producer, string message) => 
    {
        await producer.PublishMessageAsync(message);

        return Results.Ok();
    }
);

app.Run();
```

Le `conteneur de service` se charge de disposer proprement mon `producer`.