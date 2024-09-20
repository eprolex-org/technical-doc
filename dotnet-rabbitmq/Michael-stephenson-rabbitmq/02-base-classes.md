# 04 `.net` et `Rabbitmq`

## Les classes de base

### `IConnection`

Reprèsente une connexion à `rabbitmq` utilisant `AMQP`.

### `IModel`

Reprèsente un canal (`channel`) vers le serveur `AMQP` au travers duquel on peut réaliser des `actions` : `créer une queue`, `envoyer un message`, ...

### `ConnectionFactory`

Permet de construire un objet `IConnection`.

### `QueueingBasicConsumer`

C'est le moyen le plus commun pour recevoir des `messages`.

### `protocols`

Permet de choisir quelle version d'`AMQP` on veut utiliser.



## Créer une `connection`

```cs
var factory = new ConnectionFactory() { HostName = "localhost" };
using var connection = factory.CreateConnection();
```

> Par défaut on a les crédentials suivant :
>
> ```cs
> // Default password (value: "guest").
> public const string DefaultPass = "guest";
> 
> // Default user name (value: "guest").
> public const string DefaultUser = "guest";
> ```



## Créer un `channel`

```cs
using var channel = connection.CreateModel();
```

Méthodes courantes de `channel` :

- `BasicPublish` pour envoyer un message
- `ExchangeDeclare`
- `QueueBind` lie une `queue` à un `exchange`
- `QueueDeclare`

