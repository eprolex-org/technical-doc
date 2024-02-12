# 07 Créer une `Session`

> une **session** est une période délimitée pendant laquelle un appareil informatique est en communication et réalise des opérations au service d'un client - un usager, un logiciel ou un autre appareil.



## Ajout d'une `Session` 

La `Session` est gérée par les `Cookies`, on va stocker le `jwt` dans un `Cookie`.

### Ajout su service : `Sevices.AddSession`

```cs
builder.Services.AddSession(options =>
{
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
    options.IdleTimeout = TimeSpan.FromMinutes(30);
});
```

`options.Cookie.HttpOnly = true` détermine si le `Cookie` est accessible à un script (`js`), `true` signifie qu'il n'est pas accessible.

` options.Cookie.IsEssential = true` le `Cookie` peut être envoyé sans le consentement de l'utilisateur.

`options.IdleTimeout` donne le temps effectif de la `Session`, `idle` voulant dire `inactif`.



### Ajout d'un middleware `UseSession`

```c#
app.UseAuthentication();
app.UseAuthorization();

app.UseSession(); // <- ici

app.MapRazorPages();
```

