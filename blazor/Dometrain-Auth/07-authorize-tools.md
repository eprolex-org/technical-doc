# 07 Les outils pour l'`Authorization`

##  `AuthorizeRouteView`

> Fourni les mêmes fonctionnalités qu'avec `AuthenticationStateProvider` en plus simple et plus lisible.
>
> (évite le problème de la `custom implémentation`)

```html
<Router AppAssembly="typeof(Program).Assembly">
    <Found Context="routeData">
        <AuthorizeRouteView RouteData="routeData" DefaultLayout="typeof(Layout.MainLayout)">
            <NotAuthorized>
                <RedirectToLogin/>
            </NotAuthorized>
        </AuthorizeRouteView>
        <FocusOnNavigate RouteData="routeData" Selector="h1"/>
    </Found>
</Router>
```

On voit ici que les `pages` non autorisées sont redirigée vers `Login` grâce au composant `RedirectToLogin`.



## `AuthorizeView`

```ruby
@page "/auth-view"
<h3>AuthView</h3>

<AuthorizeView Roles="Boulanger">
    <p>tu es boulanger</p>
</AuthorizeView>

<AuthorizeView Policy="SupervisorOnly">
    <p>tu es supervisor</p>
</AuthorizeView>
```

Une manière super simple de créer des zones sous autorisation dans le `template`.

```html
<AuthorizeView>
    
	<Authorized>
    	<p>Authorized</p>
    </Authorized>
    
    <NotAuthorized>
    	<p>Not Authorized</p>
    </NotAuthorized>
    
    <Authorizing>
    	<p>Authorizing ...</p>
    </Authorizing>
    
</AuthorizeView>
```

On peut mettre un `spinner` dans `<Authorizing>` si le service d'autorisation effectue un `call` pouvant prendre un certain temps.

On peut aussi utiliser une `resource` liée à une `policy` :

```html
<AuthorizeView Policy="ItemAllowed" Resource="@item">
    <p>@item.Name : @item.Description is allowed</p>
</AuthorizeView>

@code {
    MyItem item = new("ingredient", "farine");
}
```

Voire les `requirements` basés sur une `resource`.



## L'attribut `[Authorize]`

```ruby
@attribute [Authorize(Policy="SupervisorOnly", Roles="Supervisor")]
```



## Exemple : un bouton `autorisé`

`AuthButton.razor`

```ruby
<AuthorizeView Policy="@Policy" Roles="@Roles">
    
    <Authorized>
        <button class="btn btn-secondary" @onclick="@Onclick">
            @ChildContent
        </button>
    </Authorized>
    
    <NotAuthorized>
        <button class="btn btn-secondary" disabled>
            @ChildContent
        </button>
    </NotAuthorized>
    
</AuthorizeView>


@code {

    [Parameter] public EventCallback Onclick { get; set; }
    [Parameter] public RenderFragment? ChildContent { get; set; }
    [Parameter] public string? Policy { get; set; }
    [Parameter] public string? Roles { get; set; }
    
}
```

Utilisation :

```ruby
<AuthButton Onclick="IncrementCount" Roles="Boulanger">
    Click me
</AuthButton>

@code {
    private int currentCount = 0;

    private void IncrementCount()
    {
        currentCount++;
    }

}
```

