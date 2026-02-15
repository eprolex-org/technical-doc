# DD Architecture SignalR dans Eprolex

## 1. Conventions de nom

Un client réagit à un `event` tandis qu'un `hub` exécute une `method`.

On a aussi besoin du nom des `Groups`.

Tous ces noms sont rangé dans des `enum` :

```cs
public enum AppMethods
{
    AddToGroup,
    RemoveFromGroup
}
```

```cs
public enum DelegueClientEvents
{
    DelegueCreated,
    DelegueDeleted,
    DelegueUpdated
}
```

Il faudrait une archi :

`EnumNames`

​	|_ `ClientEvents`

​	|_ `HubMethods`

​	|_ `Groups`

## 2. Les `notifications`

Les différentes infos reçus ou envoyées sont rassemblées dans des classes de notification :
```cs
public record DemandeAvisEventNotification(
    int DemandeAvisId,
    string EventName,
    string? ConnectionId,
    string? GroupName,
    string Prenom,
    string Nom
);
```

