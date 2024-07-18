# 04 Les `Groups`

## `Groups` définition

On peut vouloir aussi communiquer juste avec un groupe des utilisateurs.

Un `Connection Id` est créé à chaque `Connection` pour un `User`. Ce `Connection Id` change à chaque fois que la connexion est interrompu et reprise (coupure réseau, `F5`).

Un `Group` existe aussi longtemps qu'il possède au moins un `Connection Id`.



## Ajouter dans un `Group` depuis le `Hub`

```cs
public class DocumentHub : Hub
{
    public async Task AddToGroup(string demandeId)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, demandeId);
    }
```

Cette méthode pourrait-être appelée par un `Client` à l'arrivée sur une page de cette manière :

```cs 
// Un composant Blazor

protected override async Task OnInitializedAsync()
{
    await Connection.SendAsync("AddToGroup", groupName); // groupName = demandeId
```

### Utilisation du `Group` dans le `Hub`

```cs
await Clients.OthersInGroup(groupName).SendAsync(DocumentEvent.ChangeNotifyed);
```

```cs
await Clients.Group(groupName).SendAsync("SomethingToDo");
```



## Ajouter dans un `Group` depuis un `Endpoint`

```cs
await context.Groups.AddToGroupAsync(connectionId, groupName);
```

### Utilisation du `Group` dans le `Endpoint`

```cs
await context.Clients.GroupExcept(groupName, connectionId)
    .SendAsync("ChangeNotified");
```



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

    await context.Clients.Client(connectionId).SendAsync(DocumentEvent.Added, document);
    await context.Clients.GroupExcept(groupName, connectionId).SendAsync(DocumentEvent.ChangeNotifyed);
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

    await context.Clients
        .Client(connectionId)
        .SendAsync(DocumentEvent.Deleted, documentId);
    await context.Clients
        .GroupExcept(Convert.ToString(documentToDelete.DemandeId), connectionId)
        .SendAsync(DocumentEvent.ChangeNotifyed);

    return NoContent();
});
```

