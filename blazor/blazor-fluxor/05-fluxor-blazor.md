# 05 Utilisation de `Fluxor` avec `Blazor`

## Ajouter le `package`

```bash
dotnet add package Fluxor.Blazor.Web
```

Si on veut les outils `Chrome` de `Redux` :

```bash
dotnet add package Fluxor.Blazor.Web.ReduxDevTools
```



## Mise en place dans `Program.cs`

```cs
using Fluxor;

// ...

builder.Services.AddFluxor(options =>
{
 options
   .ScanAssemblies(typeof(Program).Assembly)
   .UseReduxDevTools();
});
```



## Initialiser le `Store`

Il faut ajouter une balise spécial dans `App.razor`:

```html
<Fluxor.Blazor.Web.StoreInitializer/>

<Router AppAssembly="@typeof(App).Assembly">
    // ...
```

Ou

```ruby
@using Fluxor.Blazor.Web
```

```html
<StoreInitializer />

<Router AppAssembly="@typeof(App).Assembly">
    // ...
```



## Re-Rendu du `Component`

Pour être sûr que le composant soit de nouveau rendu au changement du `state`, il faut hériter de `FluxorComponent`:

```ruby
@inherits Fluxor.Blazor.Web.Components.FluxorComponent
```

On peut aussi mettre `@using Fluxor.Blazor.Web.Components` dans `_Imports.razor`:

```ruby
@using Fluxor.Blazor.Web.Components
```

Et simplement hérité comme ceci dans le `composant` :

```ruby
@inherits FluxorComponent
```



## Un composant `Fetch`

```ruby
@page "/fetchdata"

@inherits FluxorComponent

@using Fluxor
@using FluxorBlazorDocumentationDemo.Services
@using FluxorBlazorDocumentationDemo.Store.EmployeeUseCase

@if (State.Value.IsLoading)
{
    <p>
        <em>Loading...</em>
    </p>
}
else
{
    <table class="table">
        <thead>
        <tr>
            <th>First Name</th>
            <th>Last Name</th>
        </tr>
        </thead>
        <tbody>
        @foreach (var employee in State.Value.Employees)
        {
            <tr>
                <td>@employee.FirstName</td>
                <td>@employee.LastName</td>
            </tr>
        }
        </tbody>
    </table>
}

@code {
    [Inject]
    public IState<EmployeeState> State { get; set; } = default!;

    [Inject]
    public IDispatcher Dispatcher { get; set; } = default!;

    protected override void OnInitialized()
    {
        Dispatcher.Dispatch(new GetAllEmployeesAction());
        
        base.OnInitialized();
    }
}
```

> ## ! `base.OnInitialized()` est obligatoire, sinon on a une `exception`.

### Règles à suivre

1. Hériter de `FluxorComponent` :  `@inherits FluxorComponent`

2. Injecter `IState` et `IDispatcher`

   ```cs
   [Inject]
   public IState<EmployeeState> State { get; set; } = default!;
   
   [Inject]
   public IDispatcher Dispatcher { get; set; } = default!;
   ```

   Ou avec `@inject` dans la partie `template`.

   ```html
   @inject IState<EmployeeState> State
   @inject IDispatcher Dispatcher
   ```

3. Utiliser `State.Value.IsLoading` :
   ```ruby
   @if (State.Value.IsLoading)
   {
       // ... loading
   ```

4. Utiliser `State.Value.Employees`
   ```ruby
   @foreach (var employee in State.Value.Employees)
   {
       // ...
   ```

5. Utiliser `Dispatch`, si c'est dans `OnInitialized`, oublier `base.OnInitialized()` provoque une erreur.
   ```cs
   protected override void OnInitialized()
   {
       Dispatcher.Dispatch(new GetAllEmployeesAction());
   
       base.OnInitialized();
   }
   ```

   



