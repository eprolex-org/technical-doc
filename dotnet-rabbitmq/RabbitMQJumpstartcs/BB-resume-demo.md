# BB Résumé pour la démo

## Terminologie

`Message Broker` distribué = courtier (intermédiaire) en message sur le réseau.

Communication `asynchrone`.

<img src="assets/terminologie-rabbit-mq-producer-co%20summer.png" alt="terminologie-rabbit-mq-producer-co summer" />

`Producer` = émetteur, il publie : `Publishing` un `message`.

`Consumer` = destinataire du `message`.

> Une fois le `message` envoyé, on n'attend pas la réponse. (≠ call `HTTP`)

`Exchange` = centre de trie, routeur de `message`. Il existe plusieurs types d'`exchange`.

`Binding` = un lien entre une `queue` et un exchange.

Une `queue` peut être `Bindée` à plusieurs `exchanges`.

Un `exchange` peut être `bindé` à plusieurs `queues`.

Un `consumer` peut consommer (s'abonner) à plusieurs `queues`.

Une `connection` = c'est une connexion `TCP` entre l'application et le serveur `RabbitMQ`. Elle reste ouverte pendant toute la durée de vie de l'application.

Les `Channels` = canaux, `thread` séparé pour exécuter une opération. Plusieurs `channels` par `connection` pour séparer logiquement les tâches ou effectuer des opérations concurrentes.



## Exemple simple

<img src="assets/default-simple-example-rabbit-mq.png" alt="default-simple-example-rabbit-mq" />

### Exo 1

 créer un exemple simple avec deux applications `console`. (30mn)

=> Corrrection Code démo.



## `AMQP` : Advanced Message Queuing Protocole

`RabbitMQ` utilise la version `0.9.1`. (la version `1.0` ets complétement différente)



## `Consumers` concurrents

Cela permet de ne pas bloquer la fenêtre de communication `HTTP`.

Cela améliore la `scalability` et la `reliability` (fiabilité).

<img src="assets/co%20sumer-worker-concurrent-when-one-crasj-ddd.png" alt="co sumer-worker-concurrent-when-one-crasj-ddd" />

Si un `worker` crash, il reste un autre `worker` pour accomplir la tâche.

### Exo 2

Créer un exemple avec un `Producer` et deux `Worker` (Console App)



## Accusé de reception

Avec `autoAck: true`, en cas de problème les `messages` ne sont pas gardés dans la `Queue`.

Si l'accusé de reception manuel n'est pas renvoyé, le message retourne dans la `Queue`.



### Exo 3

Ajouter les accusé de reception manuels



## `Round-robin` : chacun son tour

Les `messages` sont redistribués de manière équitable entre les `workers`.

### Problème

Un `worker` ayant fini reste à attendre.

## Solution : `prefetch`

### Exo 4

On va traiter les messages un par un en configurant le `channel` des `workers` (`consumers`).

Un `channel` peut être configuré grâce à sa méthode `BasicQoS`, Quality of Service.

> ### Remarque :
>
> Si un des `Worker` crash seul un `message` reste bloqué jusqu'à ce que celui-ci s'arrête ou redémarre



## `Pub`-`Sub` pattern

Le `Publisher` ne s'occupe pas de quel `consumer` va recevoir le `message`. Chaque consumer se `bind` lui même au `Fanout Exchange`. Le `Fanout Exchange` envoie une copie du `message` à n'importe quel `consumer` s'étant `bindé` avec lui.

### déclaration du `Fanout Exchange` dans le `Publisher`

```cs
channel.ExchangeDeclare(exchange: "user_created", type: ExchangeType.Fanout);
```

### `Binding` dans un `consumer`

```cs
channel.QueueBind(queue: queueName, exchange: "user_created", routingKey: "");
```



## `Async` Consumer

// à faire



## [Bonus] le `routing`



## [Bonus] `Requset`-`Response` pattern



