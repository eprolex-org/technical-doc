# BB Error `queue`

L'idée c'est de sortir les messages lançants des `exceptions` de la `queue` principale et de les déplacer dans une `error-queue`.



## Déclaration d'une `error-queue`

```cs
await channel.QueueDeclareAsync(
    queue: "error-queue",
    durable: true,
    autoDelete: true,
    exclusive: false
);
```

La `queue` disparait automatiqument lorsqu'elle est vide.



## `Consumer` gérant une `error-queue`

```cs
consumer.ReceivedAsync += async (sender, ea) => {

    var innerChannel = ((AsyncEventingBasicConsumer)sender).Channel;

    var body = ea.Body.ToArray();
    var messageStr = Encoding.UTF8.GetString(body);
    var message = JsonSerializer.Deserialize<MyMessage>(messageStr) ?? new MyMessage(0, "");

    try {
        // Traitement pouvant générer une exception
    }
    catch (Exception e) {
        Console.WriteLine(e.Message);
    
        await innerChannel.BasicPublishAsync(
            exchange: "",
            routingKey: "error-queue",
            body: ea.Body
        );
        
        await innerChannel.BasicAckAsync(
            deliveryTag: ea.DeliveryTag,
            multiple: false
        );
    
        throw;
    }

    await innerChannel.BasicAckAsync(
        deliveryTag: ea.DeliveryTag,
        multiple: false
    );
};
```

Pour ne pas utiliser le `channel` déclaré dans la `Thread` principale, on utilise `((AsyncEventingBasicConsumer)sender).Channel`.

Dans le `catch` :

1. On envoie le message problématique à `error-queue`.
2. On accuse réception du `message` pour le supprimer de la `queue` principale.

