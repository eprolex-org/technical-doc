# 05. `Authorize View`

## Usage de base `<AuthorizeView>`

```xml
<AuthorizeView>
    <MudText Typo="Typo.h4" Color="Color.Tertiary">
        Seulement si tu es connecté sinon rien
    </MudText>
</AuthorizeView>
```

Cette partie est accessible uniquement aux utilisateurs authentifiés.



## `<Authorized>` et `<NotAuthorized>`

Permet d'afficher du contenu suivant que l'utilisateur est authentifié ou non :

```xml 
<AuthorizeView>
    
    <Authorized>
        <MudText Class="pa-4 ma-4" Typo="Typo.body1" Color="Color.Primary">
            Hello vous êtes connecté
        </MudText>
    </Authorized>
    
    <NotAuthorized>
        <MudText Class="pa-4 ma-4" Typo="Typo.body1" Color="Color.Error">
            Vous nêtes pas connecté.
        </MudText>
    </NotAuthorized>
    
</AuthorizeView>
```

On a accès au `context` à l'intérieur de `AuthorizeView`, celui-ci contient le `User`.

```xml
<AuthorizeView>
    <Authorized>
        <MudText Class="pa-4 ma-4" Typo="Typo.body1" Color="Color.Primary">
            Hello vous êtes connecté @context.User.Identity?.Name
        </MudText>
    </Authorized>
   // ...
```

Il existe aussi un tag `<Authorizing>` pour les authentification asynchrone.



## Avec les `Roles` : `<AuthorizeView Roles="Writer">`

```xml
<AuthorizeView Roles="Writer">
    
    <Authorized>
        <MudText Class="pa-4 ma-4" Typo="Typo.body1" Color="Color.Success">
            Writer Role Area
        </MudText>
    </Authorized>
    
    <NotAuthorized>
        <MudText Class="pa-4 ma-4" Typo="Typo.body1" Color="Color.Error">
            You're not a writer
        </MudText>
    </NotAuthorized>
    
</AuthorizeView>
```

Plusieurs rôles :

```xml
<AuthorizeView Roles="Writer,Reader">
```



## Avec une `Policy` : 

## `<AuthorizeView Policy="MichelineAdminPolicy">`

```xml
<AuthorizeView Policy="MichelineAdminPolicy">
    
    <MudText Class="pa-4 ma-4" Typo="Typo.body1" Color="Color.Info">
        Micheline and admin policy
    </MudText>
    
</AuthorizeView>
```

