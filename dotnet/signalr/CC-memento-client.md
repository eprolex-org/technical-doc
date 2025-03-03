# CC memento de `SignalR` Client

## Mise en place

```bash
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

```cs
await using HubConnection connection = new HubConnectionBuilder()
    .WithUrl("http://localhost:5042/viewhub").Build();
```

> Implementions `IAsyncDisposable`, on peut avoir besoin d'un appele manuel :
>
> ```cs
> await connection.DisposeAsync();
> ```



## Démarrer la `connection`

```cs
await hubConnection.StartAsync();
```

### Statut de la `connection`

```cs
hubConnection?.State
```



## S'abonner aux événements

```cs
connection.On("ReceiveMessage", () =>{ });
```

```cs
connection
    .On<int, string>("ReceiveMessage", (userId, message) =>{ });
```



## Appeler les méthodes du `Hub`

```cs
await hubConnection.SendAsync("SendMessage", arg1, arg2);
```

### récupérer une valeur

```cs
var CompleteName 
    = await hubConnection.InvokeAsync<string>("CName", FtName, LName);
```

