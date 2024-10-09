# AA Connexion `SignalR` et `Middleware` d'authentification



## Requête de négociation

`SignalR` utilise une requête `POST` pour négocier le type de technologie du canal.

```http
POST http://localhost:8080/demande-hub/negotiate?negotiateVersion=1
```



## Forcer `WebSocket`

On peut vouloir éviter la négociation et forcer `SignalR` à utiliser `WebSocket` dans les `options` de configuration du service (ou du `Hub` lui même).

Sur le `client` :

```cs
public HubConnection Connection { get;  } = new HubConnectionBuilder()
    .WithUrl("http://localhost:8080/demande-hub"
        , options =>
    {
        options.Transports = HttpTransportType.WebSockets;
        options.SkipNegotiation = true;
    } 
        )
    .Build();
```

Sur le `serveur` :

```cs
app.MapHub<DemandeAvisHub>("/demande-hub", options =>
{
    options.Transports = HttpTransportType.WebSockets;
});
```

Je n'ai pas ici `SkipNegociation`. Cela peut-être intéressant en améliorant un peu les performances en contraignant des deux côtés l'utilisation unique de `WebSocket`.

On a plus alors de requête `POST`.



## `WebSocket` requête initiale

Une requête initiale est toujours nécessaire pour établir la connexion `WebSocket` elle-même.

```http
GET ws://localhost:8080/demande-hub
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

Le `client` demande au `serveur` d'`upgrader` la requête `HTTP` en connexion `WebSocket`.

Code avec `JWT` :

```cs
var connection = new HubConnectionBuilder()
    .WithUrl("wss://localhost:8080/demande-hub", options =>
    {
        options.Transports = HttpTransportType.WebSockets;
        options.SkipNegotiation = true;
        options.AccessTokenProvider = () => Task.FromResult("YOUR_JWT_TOKEN");
    })
    .Build();

await connection.StartAsync();
```

> Voire la différence entre `wss://` et `https://` ???

### Même avec `SkipNegociation`, il y a toujours une requête `HTTP` initiale avec `GET`.
