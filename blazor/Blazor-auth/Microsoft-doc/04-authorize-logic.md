
# 04. Personnaliser le contenu non autorisé avec le composant `Router`

Le composant `Router`, en combinaison avec le composant `AuthorizeRouteView`, permet à l’application de spécifier un contenu personnalisé dans les cas suivants :

- L’utilisateur échoue à une condition `[Authorize]` appliquée au composant. Le balisage de l’élément `<NotAuthorized>` est affiché. L’attribut `[Authorize]` est décrit dans la section correspondante.
- Une autorisation asynchrone est en cours, ce qui signifie généralement que le processus d’authentification de l’utilisateur est en cours. Le balisage de l’élément `<Authorizing>` est affiché.

> ⚠️ **Important**  
> Les fonctionnalités du routeur Blazor qui affichent le contenu `<NotAuthorized>` et `<NotFound>` ne sont **pas opérationnelles** pendant le rendu statique côté serveur (SSR statique), car le traitement des requêtes est entièrement géré par le pipeline middleware d’ASP.NET Core, et les composants Razor ne sont pas du tout rendus pour les requêtes non autorisées ou incorrectes. Utilisez des techniques côté serveur pour gérer ces cas pendant le SSR statique. Pour plus d’informations, voir *ASP.NET Core Blazor render modes*.

```html
<Router ...>
    <Found ...>
        <AuthorizeRouteView ...>
            <NotAuthorized>
                ...
            </NotAuthorized>
            <Authorizing>
                ...
            </Authorizing>
        </AuthorizeRouteView>
    </Found>
</Router>
```

Le contenu de `Authorized` et `NotAuthorized` peut inclure n’importe quel élément, y compris d’autres composants interactifs.

> 💡 **Remarque**  
> Le fonctionnement ci-dessus nécessite l’enregistrement du service `CascadingAuthenticationState` dans le fichier `Program` de l’application :

```csharp
builder.Services.AddCascadingAuthenticationState();
```

Si aucun contenu `NotAuthorized` n’est spécifié, `AuthorizeRouteView` utilise le message de secours suivant :

```html
Not authorized.
```

Une application créée à partir du modèle de projet Blazor WebAssembly avec l’authentification activée inclut un composant `RedirectToLogin`, placé dans le contenu `<NotAuthorized>` du composant `Router`. Lorsque l’utilisateur n’est pas authentifié (`context.User.Identity?.IsAuthenticated != true`), le composant `RedirectToLogin` redirige le navigateur vers le point de terminaison d’authentification/connexion. L’utilisateur est ensuite redirigé vers l’URL demandée après s’être authentifié auprès du fournisseur d’identité.

---

## Logique procédurale

Si l’application doit vérifier des règles d’autorisation dans une logique procédurale, utilisez un paramètre en cascade de type `Task<AuthenticationState>` pour obtenir le `ClaimsPrincipal` de l’utilisateur. Ce `Task<AuthenticationState>` peut être combiné avec d’autres services comme `IAuthorizationService` pour évaluer des stratégies.

Dans l’exemple suivant :

- `user.Identity.IsAuthenticated` exécute du code pour les utilisateurs connectés.
- `user.IsInRole("Admin")` exécute du code pour les utilisateurs ayant le rôle `Admin`.
- `(await AuthorizationService.AuthorizeAsync(user, "content-editor")).Succeeded` exécute du code pour les utilisateurs satisfaisant la stratégie `content-editor`.

Dans une application Blazor côté serveur, les espaces de noms nécessaires sont inclus automatiquement. Dans une application Blazor côté client, assurez-vous que les espaces de noms suivants sont présents dans le composant ou dans le fichier `_Imports.razor` :

```ruby
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.Authorization
```

### `ProceduralLogic.razor`

```ruby
@page "/procedural-logic"
@inject IAuthorizationService AuthorizationService
<h1>Exemple de logique procédurale</h1>

<button @onclick="@DoSomething">Faire quelque chose d’important</button>

@code {
    [CascadingParameter]
    private Task<AuthenticationState>? authenticationState { get; set; }

    private async Task DoSomething()
    {
        if (authenticationState is not null)
        {
            var authState = await authenticationState;
            var user = authState?.User;

            if (user is not null)
            {
                if (user.Identity is not null && user.Identity.IsAuthenticated)
                {
                    // ...
                }

                if (user.IsInRole("Admin"))
                {
                    // ...
                }

                if ((await AuthorizationService.AuthorizeAsync(user, "content-editor"))
                    .Succeeded)
                {
                    // ...
                }
            }
        }
    }
}
```

---

## Dépannage

Erreurs courantes :

- **L’autorisation nécessite un paramètre en cascade de type `Task<AuthenticationState>`.** Envisagez d’utiliser `CascadingAuthenticationState` pour le fournir.
- **Une valeur `null` est reçue pour `authenticationStateTask`.**  
  Il est probable que le projet n’ait pas été créé avec un modèle Blazor côté serveur avec l’authentification activée.

### En .NET 7 ou version antérieure

Encapsulez une partie de l’arborescence de l’interface utilisateur avec `<CascadingAuthenticationState>`, par exemple autour du routeur Blazor :

```razor
<CascadingAuthenticationState>
    <Router ...>
        ...
    </Router>
</CascadingAuthenticationState>
```

### En .NET 8 ou version ultérieure

Ne pas utiliser le composant `CascadingAuthenticationState`. À la place, ajoutez les services d’état d’authentification en cascade dans le fichier `Program` :

```csharp
builder.Services.AddCascadingAuthenticationState();
```

Le composant `CascadingAuthenticationState` (en .NET 7 ou antérieur) ou les services fournis par `AddCascadingAuthenticationState` (en .NET 8 ou ultérieur) fournissent le paramètre en cascade `Task<AuthenticationState>`, qui est lui-même fourni par le service `AuthenticationStateProvider` via l’injection de dépendances.
