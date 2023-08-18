# 02 Work `Queue`

L'idée est de distribuer des tâches lourdes (prennat beaucoup de temps, beaucoup de ressources) à des `workers`.

On encapsule la `Task` à produire dans un `Message` et on l'envoie dans la `Queue`.



## `NewTask`

```cs
var factory = new ConnectionFactory { HostName = "localhost" };

using var connection = factory.CreateConnection();
using var channel = connection.CreateModel();

channel.QueueDeclare(
    queue: "HelloHKR",
    durable: false,
    exclusive: false,
    autoDelete: false,
    arguments: null);

var message = GetMessage(args);
var bytesMessage = Encoding.UTF8.GetBytes(message);

channel.BasicPublish(exchange: string.Empty, routingKey: "HelloHKR", body: bytesMessage);

Console.WriteLine($" [x] Sent {message}");

string GetMessage(string[] args)
    => (args.Length > 0) ? string.Join(" ", args) : "Hello Hukar!";
```

`NewTask` lance une tâche simple (un `Message`) chaque fois qu'on l'exécute.



## `Worker`

```cs
var factory = new ConnectionFactory { HostName = "localhost" };

using var connection = factory.CreateConnection();
using var channel = connection.CreateModel();

channel.QueueDeclare(
    queue: "HelloHKR",
    durable: false,
    exclusive: false,
    autoDelete: false,
    arguments: null);
    
var consumer = new EventingBasicConsumer(channel);
consumer.Received += (model, ea) =>
{
    var arrBytes = ea.Body.ToArray();
    var message = Encoding.UTF8.GetString(arrBytes);

    Console.WriteLine($"message received: [{message}]");
    
    int dots = message.Split('.').Length - 1;
    Thread.Sleep(dots * 1000);

    Console.WriteLine(" [x] Done");
};

channel.BasicConsume(queue: "HelloHKR", consumer: consumer, autoAck: true);

Console.WriteLine(" Press [enter] to exit.");
Console.ReadLine();
```

Ici on va créer un `Consumer` et simuler une tâche plus ou moins longue celon le nombre de point `.` du message en reçu.

` autoAck: true` permet d'effacer le `Message` traité par le `Worker`.

## `Round-Robin` : Chacun son tour

On va lancer deux `Worker` et voir comment les `Messages` seront reçus.

Puis on va lancer plusieurs fois `NewTask`:

```bash
dotnet run "First message."
dotnet run "Second message.."
dotnet run "Third message..."
dotnet run "Fourth message...."
dotnet run "Fifth message....."
```



### `Worker 1`

```bash
message received: [First message.]
 [x] Done
message received: [Third message...]
 [x] Done
message received: [Fifth message.....]
 [x] Done
```



### `Worker 2`

```bash
message received: [Second message..]
 [x] Done
message received: [Fourth message....]
 [x] Done
```

On voit que les `Messages` sont repartis de manière équitable entre les deux `Worker`.

#### Le comportement par défaut est `Chacun son tour` : `Round-robin`.



## `Message Acknowledgement` : accusé de reception du `Message`

Si un `Worker` tombe, on ne veut pas que le `Message` soit perdu.

Pour le moment quand `RabbitMQ` délivre un `Message`, il le marque immédiatement pour suppression.

Avec le support des `message ack`, `RabbitMQ` sait que s'il ne reçoit pas d'`ack`, il doit remettre le `Message` dans la `Queue` : `Re-Queuing`.

Le `timeout` est de `30 mn` par défaut.

Si un autre `Worker` est disponible, il recevra alors le `Message`.



## `Message Ack` Manuel

`autoAck` était pour l'instant à `true`. On va le passer à `false` et gérer l'`ack` manuellement.

```cs
channel.BasicConsume(
    queue: "HelloHKR", 
    consumer: consumer, 
    autoAck: false);
```

Ajout manuel de l'`ack`:

```cs
consumer.Received += (model, ea) =>
{
    // ...
    Thread.Sleep(dots * 1000);

    Console.WriteLine(" [x] Done");
    
    channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);
};
```



## `Message` Durability

On veut préserver les `Messages` dans le cas où `rabbitMQ` crash ou s'arrête.

### On doit déclarer la `Queue` comme `durable` 

```cs
channel.QueueDeclare(
	queue: "hello",
    durable: true,
    exclusive: false,
    autoDelete: false,
    arguments: null
);
```

> On ne peut pas déclarer deux `Queue` avec le même nom et des paramètres différents.



### On doit marquer les `Messages` comme persistent

```cs
var body = Encoding.UTF8.GetBytes(message);

var properties = channel.CreateBasicProperties();
properties.Persistent = true;
```













