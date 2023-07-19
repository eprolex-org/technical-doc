# 01 Les concepts de base

## `Redux` schéma

<img src="assets/my-redux-schema.png" alt="my-redux-schema" />



## `State`, `Actions` et `Reducers`



### Mise en place

On crée une `application console`.

On ajoute le package `Fluxor`

```bash
dotnet add package Fluxor
```

On va activer l'injection de dépendance et `Fluxor` dans `Program.cs`

```cs
var services = new ServiceCollection();

services.AddScoped<App>();

services.AddFluxor(options => options.ScanAssemblies(typeof(Program).Assembly));

IServiceProvider serviceProvider = services.BuildServiceProvider();

var app = serviceProvider.GetRequiredService<App>();

app.Run();
```

> Il faut ajouter le package `Microsoft.Extensions.DependencyInjection` pour avoir accès à `BuildServiceProvider`.
>
> Apparemment ce package n'est pas installé avec `Fluxor`.

`Fluxor` charge avec lui `Microsoft.Extensions.DependencyInjection.Abstractions `

Il scanne l'`assembly` pour trouver le code relié à `Fluxor`: `States`, `Reducers`, etc.

On crée la classe `App`:

```cs
public class App
{
    public void Run()
    {
        
    }
}
```



### Injection du `Store` dans `App`

```cs
public class App
{
    private readonly IStore _store;

    public App(IStore store)
    {
        _store = store;
    }
    
    public void Run() {
        Console.Clear();
        Console.WriteLine("Initializing Store");
        Store.InitializeAsync().Wait();
    }
}
```



### `CounterState`

Dans `/Store/CounterUseCase/CounterState.cs`

```cs
[FeatureState]
public class CounterState
{
    
}
```

