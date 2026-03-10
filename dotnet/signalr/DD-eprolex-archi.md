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

`SignalRNames`

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



## 3. `Hub` et `HubNotifier`

Le `Hub` contient des méthodes exécutables à distance par le `client`.

J'ai choisi de sortir le chemin inverse et quand le `Hub` a besoin de prévenir les `clients` d'exécuter une méthode, j'utilise un `HubNotifier` pour prévenir le `client`. 

#### Le `HubNotifier` sert à prévenir un groupe de `clients` d'un événement.



















































