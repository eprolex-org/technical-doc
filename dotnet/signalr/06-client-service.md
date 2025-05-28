# 06 Un service client `SignalR`

Création d'un service avec le `client` de `SignalR`. Par exemple pour envoyer des `notifications` depuis un `Worker Service`

```cs
using Microsoft.AspNetCore.SignalR.Client;

public class SignalRClientService : IAsyncDisposable
{
    private readonly HubConnection _connection = new HubConnectionBuilder()
        .WithUrl("http://localhost:5216/my-hub")
        .WithAutomaticReconnect()
        .Build();

    private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1);

    public SignalRClientService()
    {
        _connection.Reconnected += connectionId =>
        {
            Console.WriteLine($"signalr reconnected connectionId:{connectionId}");
        
            return Task.CompletedTask;
        };
        
        _connection.Reconnecting += e =>
        {
            Console.WriteLine($"signalr is reconnecting exception message:{e?.Message}");
        
            return Task.CompletedTask;
        };
        
        _connection.Closed += e =>
        {
            Console.WriteLine($"signalr is closed exception message:{e?.Message}");

            return Task.CompletedTask;
        };
    }

    private async Task EnsureSignalRConnectionStarted()
    {
        await _semaphore.WaitAsync();

        try
        {
            if (_connection.State == HubConnectionState.Disconnected)
            {
                await _connection.StartAsync();
                Console.WriteLine("SignalR is started");
            }
        }
        finally
        {
            _semaphore.Release();
        }
    }

    public async Task SendNotification(MyNotification notification)
    {
        await EnsureSignalRConnectionStarted();

        await _connection.SendAsync("HubMethod", notification);
    }

    public async ValueTask DisposeAsync()
    {
        await _connection.DisposeAsync();
        _semaphore.Dispose();
    }
}
```

`WithAutomaticReconnect` important pour se reconnecter en cas de micro-coupure.

> Une **sémaphore** est une **variable spéciale** (souvent un compteur) utilisée pour contrôler le **nombre de threads** ou de processus qui peuvent accéder à une **ressource limitée**.

`SemaphoreSlim` cette classe permet de mettre un compteur d'accès sur l'exécution d'un code par une `thread`. 

#### `new SemaphoreSlim(initialCount, maxCount)`

On `loggue` les différents états de connexion : `Reconnected`, `Reconnecting` et `Closed`.

Une seule méthode utilisant une `semaphore`, permet de démarrer la connexion à `SignalR` : 

`EnsureSignalRConnectionStarted`.

Cette méthode est systématiquement appelée lorsqu'on veut envoyer une `notification` :

- Si on a une connexion elle ne fait rien
- Sinon elle se reconnecte