# 05 Authentification avec un `Cookie`

Maintenant que le `Cookie` est créé, il faut un `middleware` pour le lire.

<img src="assets/adding-authentication-middleware-to-the-pipeline.png" alt="adding-authentication-middleware-to-the-pipeline" style="zoom:50%;" />

En l'absence du `middleware`, malgré la génération d'un `Cookie`, si on regarde `User.Identity` :

<img src="assets/authenticate-is-not%20-authenticatde.png" alt="authenticate-is-not -authenticatde" style="zoom:33%;" />

On voit que `IsAuthenticated: false`.

Il suffit d'ajouter le `middleware` `UseAuthentication()` :

```cs
app.UseRouting();

app.UseAuthentication();
//app.UseAuthorization();

app.MapRazorPages();
```

<img src="assets/is-auth-true-very-good.png" alt="is-auth-true-very-good" style="zoom:33%;" />

> Ce n'est pas la peine d'ajouter explicitement `UseAuthentication`, ce `middleware` est automatiquement ajouté lorsque `builder.Build()` s'exécute.
>
> Voire cette discussion : https://github.com/dotnet/aspnetcore/issues/48469



## Le `header` de la requête

Le `Cookie` est retourné vers le serveur dans le `header` de la requête :

<img src="assets/request-header-with-my-cookie-auth.png" alt="request-header-with-my-cookie-auth" style="zoom:33%;" />

Le `middleware` d'`Authentication` peut donc le décoder et le déserialiser.

Je récupère dans l'objet `User.Identity` les `Claims` définies dans le `Cookie` :

<img src="assets/user-identity-from-cookie-deserialized-decrypted.png" alt="user-identity-from-cookie-deserialized-decrypted" style="zoom:33%;" />



## Durée de vie d'un `Cookie`

On peut ajouter un temps d'expiration au `Cookie` en renseignant un `TimeSpan` dans les options du `Cookie` :

> Ce n'est pas la propriété `expires` du `cookie` mais un temps de vie du `ticket` contenu dans le `cookie` :
>
> Contrôle la durée de validité du `ticket` d'authentification stocké dans le `cookie` à partir du moment où il a été créé. Les informations relatives à l'expiration sont stockées dans le `ticket` du `cookie` protégé. C'est pourquoi un `cookie` expiré sera ignoré même s'il est transmis au serveur après que le navigateur aurait dû le purger.  Cette information est distincte de la valeur `Expires`, qui spécifie la durée de conservation du `cookie` par le navigateur. (doc .net)

`Program.cs`

```cs
builder.Services.AddAuthentication()
    .AddCookie(MyAuthScheme.CookieAuthScheme, options =>
    {
        options.Cookie.Name = MyAuthScheme.CookieAuthScheme;
        options.ExpireTimeSpan = TimeSpan.FromSeconds(60);
```

> Le `cookie` ne disparait pas après le temps d'expiration.

