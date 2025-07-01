# 01. Service `AuthenticationStateProvider`

`AuthenticationStateProvider` est le service sous-jacent utilisé par le composant `AuthorizeView` et les services d’authentification en cascade pour obtenir l’état d’authentification d’un utilisateur.

En général, vous n’utilisez pas directement `AuthenticationStateProvider`. Il est recommandé d’utiliser le composant `AuthorizeView` ou l’approche `Task<AuthenticationState>` décrite plus loin dans cet article. Le principal inconvénient de l’utilisation directe de `AuthenticationStateProvider` est que le composant n’est pas automatiquement notifié si les données d’état d’authentification sous-jacentes changent.

Pour implémenter un `AuthenticationStateProvider` personnalisé, consultez [État d’authentification dans ASP.NET Core Blazor](https://learn.microsoft.com/aspnet/core/blazor/security/?view=aspnetcore-8.0#authenticationstateprovider), qui inclut des conseils pour notifier les changements de l’état d’authentification de l’utilisateur.

---

## Obtenir les informations du `ClaimsPrincipal` d’un utilisateur

Le service `AuthenticationStateProvider` peut fournir les données du `ClaimsPrincipal` de l’utilisateur courant, comme illustré dans l’exemple suivant :

### `ClaimsPrincipalData.razor`

```ruby
@page "/claims-principal-data"
@using System.Security.Claims
@inject AuthenticationStateProvider AuthenticationStateProvider

<h1>Données du ClaimsPrincipal</h1>

<button @onclick="GetClaimsPrincipalData">Obtenir les données ClaimsPrincipal</button>

<p>@authMessage</p>

@if (claims.Any())
{
    <ul>
        @foreach (var claim in claims)
        {
            <li>@claim.Type: @claim.Value</li>
        }
    </ul>
}

<p>@surname</p>

@code {
    private string? authMessage;
    private string? surname;
    private IEnumerable<Claim> claims = Enumerable.Empty<Claim>();

    private async Task GetClaimsPrincipalData()
    {
        var authState = await AuthenticationStateProvider.GetAuthenticationStateAsync();
        var user = authState.User;

        if (user.Identity is not null && user.Identity.IsAuthenticated)
        {
            authMessage = $"{user.Identity.Name} est authentifié.";
            claims = user.Claims;
            surname = user.FindFirst(c => c.Type == ClaimTypes.Surname)?.Value;
        }
        else
        {
            authMessage = "L’utilisateur n’est PAS authentifié.";
        }
    }
}
```

**Explication de l’exemple :**

* `ClaimsPrincipal.Claims` retourne les revendications de l’utilisateur (claims) pour affichage dans l’interface.
* La ligne qui obtient le nom de famille (`surname`) utilise `FindFirst` avec un prédicat pour filtrer les revendications.
* Si `user.Identity.IsAuthenticated` est `true`, et que l’utilisateur est un `ClaimsPrincipal`, alors on peut énumérer les revendications et évaluer son appartenance à des rôles.

Pour en savoir plus sur l’injection de dépendances (DI) et les services, voir :

* [Injection de dépendances dans Blazor](https://learn.microsoft.com/aspnet/core/blazor/fundamentals/dependency-injection)
* [Injection de dépendances dans ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection)

---

## Exposer l’état d’authentification comme un paramètre en cascade

Si vous avez besoin de l’état d’authentification pour une logique procédurale (ex. : lors d’une action utilisateur), vous pouvez l’obtenir via un paramètre en cascade de type `Task<AuthenticationState>`, comme montré ci-dessous.

### `CascadeAuthState.razor`

```ruby
@page "/cascade-auth-state"

<h1>État d’authentification en cascade</h1>

<p>@authMessage</p>

@code {
    private string authMessage = "L’utilisateur n’est PAS authentifié.";

    [CascadingParameter]
    private Task<AuthenticationState>? authenticationState { get; set; }

    protected override async Task OnInitializedAsync()
    {
        if (authenticationState is not null)
        {
            var authState = await authenticationState;
            var user = authState?.User;

            if (user?.Identity is not null && user.Identity.IsAuthenticated)
            {
                authMessage = $"{user.Identity.Name} est authentifié.";
            }
        }
    }
}
```

Si `user.Identity.IsAuthenticated` est `true`, les revendications peuvent être énumérées et les rôles évalués.

---

## Configuration du paramètre en cascade `Task<AuthenticationState>`

Pour utiliser ce paramètre, vous devez configurer les composants `AuthorizeRouteView` et les services d’état d’authentification en cascade.

Lors de la création d’une application Blazor avec authentification via un modèle de projet, celle-ci inclut automatiquement `AuthorizeRouteView` et l’appel à `AddCascadingAuthenticationState`, comme montré ci-dessous :

```ruby
<Router ...>
    <Found ...>
        <AuthorizeRouteView RouteData="routeData" 
                            DefaultLayout="typeof(Layout.MainLayout)" />
        ...
    </Found>
</Router>
```

Dans le fichier `Program`, enregistrez les services nécessaires :

```csharp
builder.Services.AddCascadingAuthenticationState();
```

* Pour une application **Blazor WebAssembly** (client-side), ajoutez également :

```csharp
builder.Services.AddAuthorizationCore();
```

* Pour une application **Blazor Server**, les services requis pour `options` et `authorization` sont déjà présents, aucune étape supplémentaire n’est nécessaire.

---
