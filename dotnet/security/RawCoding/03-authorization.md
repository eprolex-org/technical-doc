# 03 Authorization

## Définition

Si l'`authentication` permet de savoir qui vous êtes, l'`authorization` permet de savoir s'il vous est permis d'accéder à une ressource (un `endpoint`).

Si un `passeport` représente votre `authentication`, un `claim` de type `"paseport_type"` va définir vos `authorization`.



## `401 Unauthorized` Non authentifié et `403 Forbidden` accès refusé (non authorisé)

Ajoutons d'abord un `claim` régulant l'accès à une pays :

```cs
app.MapGet("/login", async (HttpContext ctx) => {
    List<Claim> claims = [
        new Claim("usr", "hukar"),
        new Claim("passport_type", "eu") // <- ici
    ];
    
    var identity = new ClaimsIdentity(claims, authScheme);
    var user = new ClaimsPrincipal(identity);
    
    await ctx.SignInAsync(authScheme, user);
    
    return "ok";
});
```

Puis créons un `endpoint` représentant un pays : `sweden`

```cs
app.MapGet("/sweden", (HttpContext ctx) =>
{
```

On vérifie que la personne a un `passeport` :

```cs
	if (!ctx.User.Identities.Any(x => x.AuthenticationType == authScheme))
```

ou

```cs
	if (ctx.User.Identities.All(x => x.AuthenticationType != authScheme))
    {
        ctx.Response.StatusCode = 401; // UNAUTHORIZED
        return "vous ne possedez pas de passport";
    }
```

On vérifie que le `passeport` est adapté :

```cs
	if (!ctx.User.HasClaim("passport_type", "eu"))
    {
        ctx.Response.StatusCode = 403; // FORBIDDEN
        return "votre passeport ne vous permez pas d'accéder à ce pays";
    }
```

Si tout est correcte on donne l'accès :

```cs
    return "Vous êtes le bienvenue en Suède";
});
```

<img src="assets/access-denied-error-401.png" alt="access-denied-error-401" />