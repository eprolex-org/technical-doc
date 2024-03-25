# 05 `Resquest/Response` pattern

On utilise les termes `Client`/`Server` plutôt que `Producer`/`Consumer` car chaque service est à la fois `Producer` et `Consumer`.

Ce pattern défini une `Queue` pour la réponse envoyée par le `Consumer`.

La réponse est envoyée typiquement par le `Default Exchange`.

C'est le `Client` qui spécifie la `Queue` de réponse via la propriété `replyQueue` du `Message`.

Le problème est de savoir comment le `Client` va pouvoir associer la réponse à une requête particulière envoyée. S'il y a beaucoup de service abonné à la même `Request Queue` il faut un mécanisme d'identification. Un `tag` unique est créé pour chaque `Request`, le `Server` peut réutiliser ce `tag` pour identifier la `Reply`, le `correlation id` est souvent utilisé dans ce but.



## Désavantages

https://www.youtube.com/watch?v=7y2DKiyHg80&ab_channel=MarkRichards

Le service `Client` doit attendre pour avoir sa réponse :

<img src="assets/request-reply-wait-for-response-async-not.png" alt="request-reply-wait-for-response-async-not" />

La deuxième conséquence est que les services deviennent fermement couplés :

<img src="assets/tightly-coupled-servoce-schema-youtube.png" alt="tightly-coupled-servoce-schema-youtube" />

On doit lui préférer `Async Notification` car les services ne sont plus couplés. `Async Notification` emploie deux `Channel` séparés.

<img src="assets/async-notification-vs-request-reply.png" alt="async-notification-vs-request-reply" />

En conclusion le pattern `Request / Reply` ne doit être utilisé que lorsqu'on demande des données en retour pour pouvoir continuer (afficher des données traités des données) à un autre service (`Microservice`) via une `Queue`.

## Propriétés des `Messages`

`AMQP` définit un ensemble de propriété associées au `Message` :

- `Persistent` marque les `Messages` pour une `Queue Durable` à être sauvegardé
- `DeliveryMode` alternative à `Persistent`
- `ContentType` Type MIME de l'encodage, par exemple `application/json` 
- `ReplyTo` désigne la `Queue` de réponse (`Reply`)
- `CorrelationId` utilisé pour corréler les `Requête` et les `Réponse` dans le pattern `RPC`



## Idempotent

Si le `Server` plante après avoir envoyé le `Message` de réponse mais avant d'envoyer l'accusé de réception, la requête sera de nouveau exécutée et le `Client` recevra de nouveau un `Message` de réponse. Le `correlationId` sert au `Client` à détecter le `Message` doublon et idéalement le traitement du `Server` devrait être `idempotent`.



## Fonctionnement

<img src="assets/rpc-correlation-id-schema-requset-response-alphonse.png" alt="rpc-correlation-id-schema-requset-response-alphonse" />

- Au démarage le `Client` créé une `Queue` anonyme (`amq.gen.Xa2...`)
- Pour une requête `RPC` le `Client` ajoute deux `Propriétés` au `Message` : `ReplyTo` et `CorrelationId` (unique pour chaque requête)

## Client

```cs
using System.Text;
using RabbitMQ.Client;
using RabbitMQ.Client.Events;
```

```cs
var factory = new ConnectionFactory { HostName = "localhots" };

using var conn = await factory.CreateConnectionAsync();
using var channel = await conn.CreateChannelAsync();

var replyQueue = await channel.QueueDeclareAsync(queue: "replyQ", exclusive: true);
var requestQueue = await channel.QueueDeclareAsync(queue: "requestQ");

var consumer = new EventingBasicConsumer(channel);

consumer.Received += (sender, eventArgs) =>
{
    var body = eventArgs.Body.ToArray();
    var message = Encoding.UTF8.GetString(body);
    
    Console.WriteLine($"Reply received:{message}");
};

await channel.BasicConsumeAsync(queue: replyQueue.QueueName, autoAck: true, consumer: consumer);

var message = "Can I request a reply";
var body = Encoding.UTF8.GetBytes(message);

await channel.BasicPublishAsync("", requestQueue.QueueName, body);

Console.WriteLine("Started Client");

Console.ReadKey();
```





## Server

```cs
var factory = new ConnectionFactory() { HostName = "localhost" };

using var conn = factory.CreateConnection();
using var channel = connection.CreateModel();

channel.QueueDeclare(Queue: "requestQ", exclusive: false);

var consumer = new EventingBasicConsumer(channel);

consumer.Received += (model, ea) => {
    Console.WriteLine($"Receiced request: {ea.BasicProperties.CorrelationId}");
    
    var replyMessage = $"this is your reply: {ea.BasicProrties.CorrelationId}";
    
    var body = Encoding.UTF8.GetBytes(replyMessage);
    
    channel.BasicPublish("", ea.BasicProperties.ReplyTo, null, body);
};

channel.BasicConsume(queue: "requestQ", autoAck: true, consumer: consumer);

Console.ReadKey();
```

<img src="assets/console-display-client-server-mickey.png" alt="console-display-client-server-mickey" />
