# 03 `HubContext`

À la place d'appeler directement mon `Hub`, je peux placer la logique dans un `endpoint` et renvoyer grâce au `HubContext` un événement aux différents clients `SignalR`.



## Dans le `Server`

### Création d'un `Hub` vide

```cs
public class EnchereHub : Hub
{

}
```

Ajout dans `Program.cs`

```cs
builder.Services.AddSignalR();

app.MapHub<EnchereHub>("enchere-hub");
```



### Utilisation du `Hub` grâce à `IHubContext` dans les `Endpoints`

On peut injcter dans un `endpoint` directement le `HubContext` pour accéder au `Hub` :

```cs
(IHubContext<EnchereHub> context) => { ... }
```



### `All`, `Client(connectionId)`, `AllExcept(connectionId)`

`All` déclenche l'événement dans tous les `Clients` connectés.

`Client(connectionId)` déclenche l'événement uniquement dans le `Client` correspondant à la `connectionId`.

`AllExcept(connectionID)` déclenche l'événement dans tous les `Clients` sauf celui désigné par la `connectionId`.



## Exemples d'utilisation

### `Add`

```cs
app.MapPost("/{connectionId}", 
    (
        EnchereRepository repo, 
        Enchere enchere, 
        IHubContext<EnchereHub> context,
        string connectionId) =>
{
    repo.Add(enchere);

    context.Clients.Client(connectionId).SendAsync("enchereAdded", enchere);
    context.Clients.AllExcept(connectionId).SendAsync("changeNotifyed");
});
```

### `Delete`

```cs
app.MapDelete("/{enchereId:int}/{connectionId}", (
    EnchereRepository repo, 
    int enchereId, 
    string connectionId, 
    IHubContext<EnchereHub> context) =>
{
    repo.Delete(enchereId);

    context.Clients.Client(connectionId).SendAsync("enchereDeleted", enchereId);
    context.Clients.AllExcept(connectionId).SendAsync("changeNotifyed");
});
```

### `Update`

```cs

```

