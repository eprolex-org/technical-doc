# 04 `IActionSubscriber`

`IActionSubscriber` vous permet de vous abonner au `Dispatch pipeline` et d'être notifié lorsqu'une `Action` est `Dispatchée`.

> Cela peut servir aussi à ne pas hériter de `FluxorComponent`, car on ne peut hériter qu'une fois.
>
> On gère alors à la main `StateHasChanged`.



## Exemple de `IActionSubscriber`

```cs
public class SomeClass : IDisposable
{
    private readonly IActionSubscriber _actionSubscriber;

    public SomeClass( IActionSubscriber actionSubscriber)
    {
        _actionSubscriber = actionSubscriber;
    }

    public void SomeMethod()
    {
        _actionSubscriber.SubscribeToAction<SomeAction>(
            this,
            action =>
            {
                // Do Something with the Action
            });

        // ...
    }

    public void Dispose()
    {
        _actionSubscriber.UnsubscribeFromAllActions(this);
    }
}
```

Voici comment une `classe` peut s'abonner au `Dispatcher`:

## `SubscribeToAction<SomeAction>`

S'abonner à une `Action`.

Cette méthode prend deux paramètre, une instance de la `classe` abonnée et une `lambda : Action<SomeAction>` donnat accès à l'`Action` `dispatchée`.



## `IDisposable` : `UnsubscribeFromAllActions`

On doit obligatoirement se désabonner en passant de nouveau l'instance de la `classe` avec `UnsubscribeFromAllActions(this)`.