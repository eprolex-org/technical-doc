# DD `Re-render` une page



## `StateHasChanged`

Pour forcer un re-rendu d'une page (un composant page), on utilise la méthode `StateHasChanged` :

```cs
public async void GetColour2()
{
    Colour2 = await httpClient.GetFromJsonAsync<Colour>(urlColour);

    StateHasChanged();
}
```



## `InvokeAsync`

Dans le cas où `StateHasChanged` est appelé d'un autre `Thread`, par exemple une `callback` avec `SignalR`, on utilise alors `InvokeAsync` :

```cs
protected override async Task OnInitializedAsync()
{
    // ...
     Provider.Connection.On("RefreshTitreConjoint", () => LoadItems());
}
```

```cs
private async Task LoadItems(bool isAsyncChanged = false)
{
    _isLoading = true;
    await InvokeAsync(StateHasChanged);

    _titresAvailable = await Repo.GetTitres(DemandeAvisId);

    _isLoading = false;
    await InvokeAsync(StateHasChanged);
}
```



## `@gestionnaire ` d'événement

```react
@if (_isInProgress)
{
    <MudProgressCircular 
        Color="Color.Info" 
        Indeterminate/>
}
else
{
    <MudButton
        Variant="Variant.Filled"
        Color="Color.Secondary"
        OnClick="ProcessSomething">
        Do Something
    </MudButton>
}
```

```cs
@code {
    private bool _isInProgress;

    private async Task ProcessSomething()
    {
        _isInProgress = true;

        await Task.Delay(800);

        _isInProgress = false;
    }
}
```

Dans ce code tout va se passer comme si `StateHasChanged` est appelé deux fois, une fois au début de l'exécution de la `méthode` associée au `gestionnaire d'événement` et une fois à la fin.

#### ! de ne pas imbriquer le changement d'une valeur comme `_isInProgress` entre plusieurs traitements asynchrone.

```cs
private async Task ProcessSomething()
{

    await Task.Delay(300);

    await Task.Delay(500);

    _isInProgress = true;

    await Task.Delay(800);

    _isInProgress = false;
}
```

Ce code ne fonctionnera pas de manière prévisible.
