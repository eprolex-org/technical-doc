
# Autorisation

Après l’authentification d’un utilisateur, des règles d’autorisation sont appliquées pour contrôler ce que l’utilisateur peut faire.

L’accès est généralement accordé ou refusé selon que :

- Un utilisateur est authentifié (connecté).
- Un utilisateur appartient à un rôle.
- Un utilisateur possède une revendication (claim).
- Une stratégie (policy) est satisfaite.

Chacun de ces concepts est identique à ceux utilisés dans une application ASP.NET Core MVC ou Razor Pages. Pour plus d’informations sur la sécurité dans ASP.NET Core, consultez les articles sous **ASP.NET Core Security and Identity**.

## Composant `AuthorizeView`

Le composant `AuthorizeView` affiche sélectivement du contenu de l’interface utilisateur selon que l’utilisateur est autorisé ou non. Cette approche est utile lorsque vous devez simplement afficher des données à l’utilisateur sans utiliser son identité dans une logique procédurale.

Le composant expose une variable de contexte de type `AuthenticationState` (`@context` en syntaxe Razor), que vous pouvez utiliser pour accéder aux informations de l’utilisateur connecté :

```ruby
<AuthorizeView>
    <p>Bonjour, @context.User.Identity?.Name !</p>
</AuthorizeView>
```

Vous pouvez également fournir un contenu différent si l’utilisateur n’est pas autorisé, en combinant les paramètres `Authorized` et `NotAuthorized` :

```ruby
<AuthorizeView>
    <Authorized>
        <p>Bonjour, @context.User.Identity?.Name !</p>
        <p><button @onclick="HandleClick">Bouton réservé aux utilisateurs autorisés</button></p>
    </Authorized>
    <NotAuthorized>
        <p>Vous n’êtes pas autorisé.</p>
    </NotAuthorized>
</AuthorizeView>

@code {
    private void HandleClick() { ... }
}
```

Bien que le composant `AuthorizeView` contrôle la visibilité des éléments selon le statut d’autorisation de l’utilisateur, il **n’applique pas la sécurité** sur le gestionnaire d’événements lui-même. Pour garantir la sécurité au niveau de la méthode, implémentez une logique d’autorisation supplémentaire dans le gestionnaire ou dans l’API concernée.

Les composants Razor des applications Blazor Web **ne** montrent jamais le contenu de `<NotAuthorized>` lorsque l’autorisation échoue côté serveur pendant le rendu statique côté serveur (SSR statique). Le pipeline ASP.NET Core côté serveur traite l’autorisation. Utilisez des techniques côté serveur pour gérer les requêtes non autorisées.

> ⚠️ **Avertissement**  
> Le balisage et les méthodes côté client associés à un `AuthorizeView` ne sont protégés que dans l’interface utilisateur rendue. Pour protéger le contenu autorisé et sécuriser les méthodes dans Blazor côté client, le contenu est généralement fourni par un appel à une API Web sécurisée et autorisée, et **jamais stocké dans l’application**.

Le contenu de `Authorized` et `NotAuthorized` peut inclure n’importe quel élément, y compris d’autres composants interactifs.

Les conditions d’autorisation, comme les rôles ou les stratégies qui contrôlent les options de l’interface ou l’accès, sont abordées dans la section **Authorization**.

Si aucune condition d’autorisation n’est spécifiée, `AuthorizeView` utilise une stratégie par défaut :

- Les utilisateurs authentifiés (connectés) sont autorisés.
- Les utilisateurs non authentifiés (déconnectés) ne le sont pas.

Le composant `AuthorizeView` peut être utilisé dans le composant `NavMenu` (`Shared/NavMenu.razor`) pour afficher un composant `NavLink`, mais cette approche ne fait que supprimer l’élément de la sortie rendue. Elle **n’empêche pas** l’utilisateur de naviguer vers le composant. Implémentez l’autorisation séparément dans le composant de destination.

## Autorisation basée sur les rôles et les stratégies

Le composant `AuthorizeView` prend en charge l’autorisation basée sur les rôles ou les stratégies.

Pour l’autorisation basée sur les rôles, utilisez le paramètre `Roles`. Exemple :

```ruby
<AuthorizeView Roles="Admin, Superuser">
    <p>Vous avez une revendication de rôle 'Admin' ou 'Superuser'.</p>
</AuthorizeView>
```

Pour exiger que l’utilisateur ait **les deux** rôles `Admin` et `Superuser`, imbriquez les composants :

```ruby
<AuthorizeView Roles="Admin">
    <p>Utilisateur : @context.User</p>
    <p>Vous avez le rôle 'Admin'.</p>
    <AuthorizeView Roles="Superuser" Context="innerContext">
        <p>Utilisateur : @innerContext.User</p>
        <p>Vous avez les rôles 'Admin' et 'Superuser'.</p>
    </AuthorizeView>
</AuthorizeView>
```

Pour l’autorisation basée sur les stratégies, utilisez le paramètre `Policy` :

```ruby
<AuthorizeView Policy="Over21">
    <p>Vous satisfaites la stratégie 'Over21'.</p>
</AuthorizeView>
```

Pour exiger plusieurs stratégies :

```ruby
<AuthorizeView Policy="Over21">
    <AuthorizeView Policy="LivesInCalifornia">
        <p>Vous satisfaites les stratégies 'Over21' et 'LivesInCalifornia'.</p>
    </AuthorizeView>
</AuthorizeView>
```

L’autorisation basée sur les revendications est un cas particulier de l’autorisation basée sur les stratégies.

Si `Roles` et `Policy` sont définis, l’autorisation réussit **uniquement si les deux conditions sont remplies**.

> ℹ️ Les comparaisons de chaînes dans .NET sont sensibles à la casse. Par exemple, `Admin` ≠ `admin`.

Les noms de rôles et de stratégies utilisent généralement la casse Pascal (`BillingAdministrator`), mais d’autres styles sont autorisés (camelCase, kebab-case, snake_case). Les espaces sont rares mais valides (`billing administrator`).

## Contenu affiché pendant l’authentification asynchrone

Blazor permet de déterminer l’état d’authentification de manière asynchrone, notamment dans les applications Blazor côté client.

Pendant l’authentification, `AuthorizeView` n’affiche aucun contenu. Pour afficher du contenu pendant ce processus, utilisez le paramètre `Authorizing` :

```ruby
<AuthorizeView>
    <Authorized>
        <p>Bonjour, @context.User.Identity?.Name !</p>
    </Authorized>
    <Authorizing>
        <p>Ce contenu est visible uniquement pendant l’authentification.</p>
    </Authorizing>
</AuthorizeView>
```

Cette approche ne s’applique généralement pas aux applications Blazor côté serveur, qui connaissent l’état d’authentification dès qu’il est établi.
