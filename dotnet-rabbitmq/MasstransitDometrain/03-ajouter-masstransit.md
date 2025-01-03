# 03 Ajouter `Masstransit`



## Le `package`

```bash
dotnet add package MassTransit.RabbitMQ
```



## Configurer le `middleware`

> Un `bus` (implémentant `IBus`) est une abstraction représentatnt la communication entre un `Producer` et un `Consumer` de `message`.
>
> Il utilise les `exchanges` et les `queues` de `RabbitMQ` pour router les `messages`.
>
> Il peut :
>
> - `Publish` : publier (pattern `pub/sub`) à tous les `Consumers`.
> - `Send` : envoyer (pattern `point-to-point`) à un destinataire spécifique.
>
> ### Interface principale
>
> ```cs
> Task Send<T>(T message, CancellationToken ct);
> Task Publish<T>(T message, CancellationToken ct);
> ```
>
> Il permet aussi de recevoir les `messages`.

`Program.cs`

```cs
builder.Services.AddMassTransit(x =>
{
    x.UsingRabbitMq((context, cfg) =>
    {
        cfg.ConfigureEndpoints(context);
    });
});
```

`ConfigureEndpoints` est le minimum à configurer pour que `MassTransit` fonctionne `out of the box`.

On peut aussi spécifier les informations du serveur `RabbitMQ` :

```cs
x.UsingRabbitMq((context, cfg) =>
{
    cfg.Host("rabbitmq://localhost:5672", "/", host =>
        {
            host.Username("guest");
            host.Password("guest");
        }
    );

    // ...
```

Mais si tous les paramètres sont ceux par défaut, ce n'est pas la peine.
