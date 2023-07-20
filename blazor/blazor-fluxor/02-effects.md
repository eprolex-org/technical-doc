# 02. `Effects`

Le `flux` du `state` est supposé être immutable. Le `state` est remplacé par des `Pure Function`, qui ne prennent que la valeur de leurs paramètres pour construire le nouveau `State`.

Il faut donc un mécanisme capable de gérer l'accès à d'autre source de données, comme des `web services`, et qui ensuite les `Reduce` pour construire notre `State`.

C'est un `Effect` qui va réaliser l'appel à un service avant de réduire (`Reducing`) le résultat en un `State`.



## Nouveau `State` : `CreatureState`

Dans `Store/CreatureUseCase`

`CreatureState.cs`

```cs
public record CreatureState(bool IsLoading, IEnumerable<Creature> Creatures);
```

Et une classe `CreatureFeature`:

```cs
public class CreatureFeature : Feature<CreatureState>
{
    public override string GetName() => nameof(CreatureState);

    protected override CreatureState GetInitialState() => new CreatureState(
        IsLoading: false,
        Creatures: Enumerable.Empty<Creature>());
}
```



## `FetchCreaturesAction` et `Reducers`

On va créer une `classe` (ou un `record`) vide.

```cs
public record FetchCreatureAction;
```

et une `static class` : `Reducers`:

```cs
public static class Reducers
{
    [ReducerMethod(typeof(FetchCreatureAction))]
    public static CreatureState ReduceFetchCreatureAction(CreatureState state) 
        => state with { IsLoading = false, }
}
```

