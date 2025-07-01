
## Attribut `[Authorize]`

L’attribut `[Authorize]` est disponible dans les composants Razor :

```ruby
@page "/"
@attribute [Authorize]

Vous ne pouvez voir ceci que si vous êtes connecté.
```

> ## ⚠️ **Important**  
> Utilisez `[Authorize]` uniquement sur les composants `@page` accessibles via le routeur Blazor. L’autorisation n’est effectuée que dans le cadre du routage, et non pour les composants enfants rendus dans une page. Pour autoriser l’affichage de parties spécifiques d’une page, utilisez plutôt `AuthorizeView`.

L’attribut `[Authorize]` prend également en charge l’autorisation basée sur les rôles ou sur les stratégies.

### Autorisation basée sur les rôles

Utilisez le paramètre `Roles` :

```ruby
@page "/"
@attribute [Authorize(Roles = "Admin, Superuser")]

<p>Vous ne pouvez voir ceci que si vous avez le rôle 'Admin' ou 'Superuser'.</p>
```

### Autorisation basée sur les stratégies

Utilisez le paramètre `Policy` :

```ruby
@page "/"
@attribute [Authorize(Policy = "Over21")]

<p>Vous ne pouvez voir ceci que si vous satisfaites la stratégie 'Over21'.</p>
```

Si ni `Roles` ni `Policy` ne sont spécifiés, `[Authorize]` utilise la stratégie par défaut :

- Les utilisateurs authentifiés (connectés) sont autorisés.
- Les utilisateurs non authentifiés (déconnectés) ne le sont pas.

Lorsque l’utilisateur n’est pas autorisé et que l’application ne personnalise pas le contenu non autorisé via le composant `Router`, le framework affiche automatiquement le message de secours suivant :

```html
Not authorized.
```

---

## Autorisation des ressources

Pour autoriser les utilisateurs à accéder à des ressources, transmettez les données de la route de la requête au paramètre `Resource` de `AuthorizeRouteView`.

Dans le contenu `Router.Found` pour une route demandée :

```ruby
<AuthorizeRouteView Resource="routeData" RouteData="routeData" 
    DefaultLayout="typeof(MainLayout)" />
```

Pour plus d’informations sur la manière dont l’état d’autorisation est transmis et utilisé dans la logique procédurale, consultez la section *Expose the authentication state as a cascading parameter*.

Lorsque `AuthorizeRouteView` reçoit les données de route pour la ressource, les stratégies d’autorisation ont accès à `RouteData.PageType` et `RouteData.RouteValues`, ce qui permet d’implémenter une logique personnalisée pour prendre des décisions d’autorisation.

### Exemple de stratégie personnalisée

Dans l’exemple suivant, une stratégie `EditUser` est créée dans `AuthorizationOptions` pour la configuration du service d’autorisation de l’application (`AddAuthorizationCore`) avec la logique suivante :

1. Vérifier si une valeur de route existe avec la clé `id`.
2. Si la clé existe, stocker la valeur dans `value`.
3. Dans une variable `id`, stocker `value` comme chaîne ou une chaîne vide (`string.Empty`).
4. Si `id` n’est pas vide, la stratégie est satisfaite si la valeur commence par `EMP`. Sinon, elle échoue.

Dans le fichier `Program` :

Ajoutez les espaces de noms :


```csharp
using Microsoft.AspNetCore.Components;
using System.Linq;
```

Ajoutez la stratégie :

```csharp
options.AddPolicy("EditUser", policy =>
    policy.RequireAssertion(context =>
    {
        if (context.Resource is RouteData rd)
        {
            var routeValue = rd.RouteValues.TryGetValue("id", out var value);
            var id = Convert.ToString(value, 
                System.Globalization.CultureInfo.InvariantCulture) ?? string.Empty;

            if (!string.IsNullOrEmpty(id))
            {
                return id.StartsWith("EMP", StringComparison.InvariantCulture);
            }
        }

        return false;
    })
);
```

> ℹ️ Cet exemple est une stratégie d’autorisation simplifiée, utilisée uniquement pour illustrer le concept avec un exemple fonctionnel. Pour plus d’informations sur la création et la configuration de stratégies, consultez *Policy-based authorization in ASP.NET Core*.

Dans le composant `EditUser` suivant, la ressource située à `/users/{id}/edit` contient un paramètre de route pour l’identifiant de l’utilisateur (`{id}`). Le composant utilise la stratégie d’autorisation `EditUser` décrite précédemment pour déterminer si la valeur de route `id` commence par `EMP`.

- Si `id` commence par `EMP`, la stratégie est satisfaite et l’accès au composant est autorisé.
- Si `id` commence par une autre valeur ou si `id` est une chaîne vide, la stratégie échoue et le composant n’est pas chargé.

#### `EditUser.razor`

```ruby
@page "/users/{id}/edit"
@using Microsoft.AspNetCore.Authorization
@attribute [Authorize(Policy = "EditUser")]

<h1>Modifier utilisateur</h1>

<p>La stratégie "EditUser" est satisfaite ! <code>Id</code> commence par 'EMP'.</p>

@code {
    [Parameter]
    public string? Id { get; set; }
}
```

