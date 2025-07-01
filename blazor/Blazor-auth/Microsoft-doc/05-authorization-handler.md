# 05. Créer ses `Handler`


## Requirements (Exigences)

Un **Requirement** (exigence) d'autorisation est un ensemble de paramètres de données qu'une **Policy** (politique) peut utiliser pour évaluer l'utilisateur actuel (le principal). Dans notre politique **"AtLeast21"**, le Requirement est un seul paramètre : l'âge minimum. Un Requirement implémente l'interface `IAuthorizationRequirement`, qui est une interface marqueur vide. Un Requirement paramétré pour l'âge minimum pourrait être implémenté comme suit :

```csharp
using Microsoft.AspNetCore.Authorization;

namespace AuthorizationPoliciesSample.Policies.Requirements;

public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public MinimumAgeRequirement(int minimumAge) =>
        MinimumAge = minimumAge;

    public int MinimumAge { get; }
}
```

Si une Policy contient plusieurs Requirements, **tous doivent être validés** pour que l’évaluation réussisse. Autrement dit, plusieurs Requirements ajoutés à une même Policy sont traités selon une logique **ET**.

> 💡 **Remarque** : Un Requirement n’a pas besoin de contenir des données ou des propriétés.

---

## Authorization Handlers (Gestionnaires d'autorisation)

Un **Handler** (gestionnaire) d'autorisation est responsable de l’évaluation des propriétés d’un Requirement. Il évalue les Requirements à l’aide d’un `AuthorizationHandlerContext` pour déterminer si l’accès est autorisé.

Un Requirement peut avoir plusieurs Handlers. Un Handler peut hériter de `AuthorizationHandler<TRequirement>`, où `TRequirement` est le Requirement à traiter. Alternativement, un Handler peut implémenter directement `IAuthorizationHandler` pour gérer plusieurs types de Requirements.

### Utiliser un Handler pour un seul Requirement

L’exemple suivant montre une relation un-à-un où un Handler d’âge minimum traite un seul Requirement :

```csharp
using System.Security.Claims;
using AuthorizationPoliciesSample.Policies.Requirements;
using Microsoft.AspNetCore.Authorization;

namespace AuthorizationPoliciesSample.Policies.Handlers;

public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, MinimumAgeRequirement requirement)
    {
        var dateOfBirthClaim = context.User.FindFirst(
            c => c.Type == ClaimTypes.DateOfBirth && c.Issuer == "http://contoso.com");

        if (dateOfBirthClaim is null)
        {
            return Task.CompletedTask;
        }

        var dateOfBirth = Convert.ToDateTime(dateOfBirthClaim.Value);
        int calculatedAge = DateTime.Today.Year - dateOfBirth.Year;
        if (dateOfBirth > DateTime.Today.AddYears(-calculatedAge))
        {
            calculatedAge--;
        }

        if (calculatedAge >= requirement.MinimumAge)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

Ce code vérifie si l’utilisateur actuel possède une revendication (`claim`) de date de naissance émise par une autorité de confiance. Si la revendication est absente, l’autorisation échoue. Si elle est présente, l’âge est calculé. Si l’utilisateur satisfait l’âge minimum requis, l’autorisation est considérée comme réussie via `context.Succeed`.

---

### Utiliser un Handler pour plusieurs Requirements
L’exemple suivant montre une relation un-à-plusieurs où un Handler de permissions traite trois types de Requirements :

```csharp
using System.Security.Claims;
using AuthorizationPoliciesSample.Policies.Requirements;
using Microsoft.AspNetCore.Authorization;

namespace AuthorizationPoliciesSample.Policies.Handlers;

public class PermissionHandler : IAuthorizationHandler
{
    public Task HandleAsync(AuthorizationHandlerContext context)
    {
        var pendingRequirements = context.PendingRequirements.ToList();

        foreach (var requirement in pendingRequirements)
        {
            if (requirement is ReadPermission)
            {
                if (IsOwner(context.User, context.Resource)
                    || IsSponsor(context.User, context.Resource))
                {
                    context.Succeed(requirement);
                }
            }
            else if (requirement is EditPermission || requirement is DeletePermission)
            {
, context.Resource))
                {
                    context.Succeed(requirement);
                }
            }
        }

        return Task.CompletedTask;
    }

    private static bool IsOwner(ClaimsPrincipal user, object? resource)
    {
        // Code omis pour la clarté
        return true;
    }

    private static bool IsSponsor(ClaimsPrincipal user, object? resource)
    {
        // Code omis pour la clarté
        return true;
    }
}
```

Ce code parcourt les Requirements en attente (`PendingRequirements`). Pour un Requirement de type `ReadPermission`, l’utilisateur doit être **propriétaire** ou **sponsor**. Pour `EditPermission` ou `DeletePermission`, il doit être **propriétaire**.

---

## Enregistrement des Handlers

Les Handlers doivent être enregistrés dans la collection de services lors de la configuration :

```csharp
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
```

Ce code enregistre `MinimumAgeHandler` en tant que singleton. Les Handlers peuvent être enregistrés avec n’importe quelle durée de vie de service intégrée.

Il est possible de **regrouper un Requirement et son Handler** dans une seule classe implémentant à la fois `IAuthorizationRequirement` et `IAuthorizationHandler`. Ce couplage est recommandé uniquement pour des Requirements simples. Cela évite d’enregistrer le Handler dans l’injection de dépendances grâce au `PassThroughAuthorizationHandler` intégré.

Consulte l’implémentation de la classe `AssertionRequirement` pour un bon exemple d’exigence auto-gérée.


## Que doit retourner un Handler ?

Notez que la méthode `Handle` dans l’exemple de Handler ne retourne aucune valeur. Alors, comment indiquer un statut de succès ou d’échec ?

Un **Handler** (gestionnaire) indique un succès en appelant `context.Succeed(IAuthorizationRequirement requirement)`, en passant le **Requirement** (exigence) qui a été validé avec succès.

Un Handler n’a généralement pas besoin de gérer les échecs, car d’autres Handlers pour le même Requirement peuvent réussir.

Pour garantir un échec, même si d’autres Handlers réussissent, appelez `context.Fail`.

Même si un Handler appelle `context.Succeed` ou `context.Fail`, **tous les autres Handlers sont quand même appelés**. Cela permet aux Requirements de produire des effets secondaires, comme de la journalisation (logging), même si un autre Handler a déjà validé ou échoué un Requirement. Lorsque la propriété `InvokeHandlersAfterFailure` est définie sur `false`, l’exécution des Handlers est interrompue dès qu’un `context.Fail` est appelé. Par défaut, `InvokeHandlersAfterFailure` est à `true`, donc tous les Handlers sont appelés.

> 💡 **Remarque** : Les Handlers d’autorisation sont appelés même si l’authentification échoue. De plus, les Handlers peuvent s’exécuter dans **n’importe quel ordre**, donc ne supposez pas un ordre d’exécution particulier.

---

## Pourquoi utiliser plusieurs Handlers pour un Requirement ?

Dans les cas où vous souhaitez une évaluation selon une logique **OU**, implémentez plusieurs Handlers pour un seul Requirement. Par exemple, Microsoft a des portes qui ne s’ouvrent qu’avec des badges. Si vous oubliez votre badge, la réceptionniste peut imprimer un autocollant temporaire et vous ouvrir la porte. Dans ce scénario, vous auriez un seul Requirement, `BuildingEntry`, mais plusieurs Handlers, chacun examinant une condition différente.

### BuildingEntryRequirement


```csharp
using Microsoft.AspNetCore.Authorization;

namespace AuthorizationPoliciesSample.Policies.Requirements;

public class BuildingEntryRequirement : IAuthorizationRequirement { }
```

### BadgeEntryHandler.cs

```csharp
using AuthorizationPoliciesSample.Policies.Requirements;
using Microsoft.AspNetCore.Authorization;

namespace AuthorizationPoliciesSample.Policies.Handlers;

public class BadgeEntryHandler : AuthorizationHandler<BuildingEntryRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, BuildingEntryRequirement requirement)
    {
        if (context.User.HasClaim(
            c => c.Type == "BadgeId" && c.Issuer == "https://microsoftsecurity"))
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

### TemporaryStickerHandler.cs

```csharp
using AuthorizationPoliciesSample.Policies.Requirements;
using Microsoft;

namespace AuthorizationPoliciesSample.Policies.Handlers;

public class TemporaryStickerHandler : AuthorizationHandler<BuildingEntryRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, BuildingEntryRequirement requirement)
    {
        if (context.User.HasClaim(
            c => c.Type == "TemporaryBadgeId" && c.Issuer == "https://microsoftsecurity"))
        {
            // Code vérifier la date d’expiration omis pour la clarté.
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

Assurez-vous que **les deux Handlers sont enregistrés**. Si l’un ou l’autre réussit lors de l’évaluation de la Policy contenant le `BuildingEntryRequirement`, alors l’évaluation de la Policy est considérée comme réussie.

---

## Utiliser une fonction (`Func`) pour satisfaire une Policy

Il peut arriver que la logique pour satisfaire une Policy possible de fournir une fonction `Func<AuthorizationHandlerContext, bool>` lors de la configuration d’une Policy avec le **Policy Builder** `RequireAssertion`.

Par exemple, le `BadgeEntryHandler` précédent pourrait être réécrit comme suit :

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("BadgeEntry", policy =>
        policy.RequireAssertion(context => context.User.HasClaim(c =>
            (c.Type == "BadgeId" || c.Type == "TemporaryBadgeId")
            && c.Issuer == "https://microsoftsecurity")));
});
