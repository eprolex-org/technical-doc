# 01 Authentification avec `Cookie`



## Authentification

C'est l'action de déterminer si quelqu'un est bien celui qu'il prétend être.

<img src="assets/cookie-first-schema-tgh.png" alt="cookie-first-schema-tgh" />

Le `cookie` contient le `ClaimsPrincipal` ainsi que quelques propriétés en plus, l'ensemble est encrypté.

L'application `asp.net` est capable de décrypter le `cookie` et retrouver les infos de `ClaimsPrincipal` :

<img src="assets/claims-principal-cookie-content-uia.png" alt="claims-principal-cookie-content-uia" />

Le `ClaimsPrincipal` est accessible via `HttpContext.User`.

<img src="assets/the-claims-principal-is-accessible-opl.png" alt="the-claims-principal-is-accessible-opl" style="zoom:33%;" />



## Implémentation dans `Program.cs`

```cs
builder.Services.AddAuthentication()
    .AddCookie(options =>
    {
        options.LoginPath = "/hukar-login";
        options.LogoutPath = "/hukar-logout";
    });

builder.Services.AddAuthorization();
```

Par défaut le `LoginPath` est `/Account/Login` et le `LogoutPah` est `/Account/Logout`.

```cs
app.UseAuthentication();

app.UseAuthorization();
```



## Implémentation des `endpoints` de `login` et de `logout`

```cs
app.MapGet("/hukar-login", async ([FromQuery] string returnUrl, HttpContext ctx) =>
        {
            Console.WriteLine($"return URL: {returnUrl}");

            var claimOne = new Claim("name", "hukar");
            var claimTwo = new Claim("password", "1234");
            var identity = new ClaimsIdentity([ claimOne, claimTwo ], CookieAuthenticationDefaults.AuthenticationScheme);

            var user = new ClaimsPrincipal(identity);
            
            await ctx.SignInAsync(user);
        });
```




> ## Problème avec `ClaimsIdentity`
>
> Si je ne spécifie pas le `authentication scheme` dans le constructeur de `ClaimsIdentity` :
>
> ```cs
> var identity = new ClaimsIdentity([ claimOne, claimTwo ]);
> ```
>
> J'obtiens l'erreur suivante :
>
> ```
> System.InvalidOperationException: SignInAsync when principal.Identity.IsAuthenticated is false is not allowed when AuthenticationOptions.RequireAuthenticatedSignIn is true.
> ```
>



## Fonctionnement

<img src="assets/sequence-diagram-login-cookie.png" alt="sequence-diagram-login-cookie" />

Le navigateur réclame une `resource` à une `api`, celle-ci le redirige (`302`) vers `/login`.

Puis `/login` retourne un cookie dans son `Response Header` :

<img src="assets/cookie-in-response-header.png" alt="cookie-in-response-header" />

On observe la redirection avec `Location: /resource` et le `cookie` dans `Set-Cookie`.

Finalement le navigateur ré-exécute la requête vers `/Resource` mais cette fois il possède le `cookie` dans son `Request Header` :

<img src="assets/request-cookie-header-tyb.png" alt="request-cookie-header-tyb" />

C'est possible car le navigateur stocke le `cookie` :

<img src="assets/cookie-stockage-browser-akt.png" alt="cookie-stockage-browser-akt" />

On peut donc observer les trois requête dans le navigateur :

<img src="assets/three-request-story-plq.png" alt="three-request-story-plq" />

> `Scalar` étant une page web effectuant les requête, comme `Postman` mais depuis le `navigateur`.





















