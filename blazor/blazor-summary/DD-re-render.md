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

