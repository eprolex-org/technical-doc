# 04 Implémentation dans un `WorkerService`

```cs
public class Worker(ILogger<Worker> logger) : BackgroundService
{
    private static readonly ushort ProcessorCount = (ushort)Environment.ProcessorCount;
    private const int K = 3;
        
    private readonly ushort prefetchCount = (ushort)(ProcessorCount * K);
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken) {

        var factory = GetFactory();
        
        await using var connection = await factory.CreateConnectionAsync(stoppingToken);
        await using var channel = await connection.CreateChannelAsync(null, stoppingToken);

        await CreateQueueAsync(channel);

        var consumer = new AsyncEventingBasicConsumer(channel);

        consumer.ReceivedAsync += HandleReceived;

        await channel.BasicConsumeAsync(
            queue: "test-concurency",
            autoAck: false,
            consumer: consumer,
            cancellationToken: stoppingToken
        );
        
        logger.LogInformation("Worker start running at: {time}", DateTimeOffset.Now);


        while (!stoppingToken.IsCancellationRequested) {
            await Task.Delay(1000, stoppingToken);
            logger.LogInformation("Worker is running : {time}", DateTimeOffset.Now);
        }

        logger.LogInformation("Worker arrêté à : {time}", DateTimeOffset.Now);
    }
    
    private static ConnectionFactory GetFactory() =>
        new()
        {
            HostName = "localhost", 
            ConsumerDispatchConcurrency = ProcessorCount
        };

    private async Task CreateQueueAsync(IChannel channel) {
        await channel.QueueDeclareAsync(
            queue: "test-concurency",
            durable: true,
            autoDelete: false,
            exclusive: false,
            arguments: null
        );

        await channel.BasicQosAsync(
            prefetchCount: prefetchCount,
            prefetchSize: 0,
            global: false
        );
    }
    
    private static async Task HandleReceived(object sender, BasicDeliverEventArgs ea) 
    {
        var body = ea.Body.ToArray();

        var message = Encoding.UTF8.GetString(body);

        Thread.Sleep(TimeSpan.FromSeconds(1));

        Console.WriteLine($"receive {message} [X]");
        
        await ((AsyncEventingBasicConsumer)sender).Channel.BasicAckAsync(ea.DeliveryTag, false);
    }
}
```

## Remarques

Comme la `connection` et le `channel` doivent être disposés proprement avec le `await using`, ils doivent être déclarés dans la méthode `ExecuteAsync` ou disposés manuellement.

Le plus simple et de ne pas créer un service séparé, mais de considérer le `worker` comme étant lui-même le service.

Comme on le voit pour le `Logger`, l'injection de dépendance est disponible dans le `worker`.