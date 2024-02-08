# 05 Protéger un `endpoint` avec un token `JWT`

## Protéger `Wheaterforecast` dans `Minimal API`

On doit seulement ajouter la méthode `RequireAuthorization` :

```cs
app.MapGet("/weatherforecast", () =>
            {
               // ...
            })
            .WithName("GetWeatherForecast")
            .WithOpenApi()
            .RequireAuthorization();
```

Pour rappel `RequireAuthorization` par défaut demande uniquement que l'utilisateur soit `authentifié`.



## Ajouter le service d'`Authorization`

Pour pouvoir utiliser `RequireAuthorization` on doit ajouter le service d'`Authorization` :

```cs
builder.Services.AddAuthorization();
```



## Configurer le service d'`authentification` par `JWT`

Dans `Program.cs` :

```cs
var secretKey = Encoding.UTF8.GetBytes(builder.Configuration["SecretKey"] ?? "");

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(secretKey),
            ValidateIssuer = false,
            ValidateAudience = false,
            ValidateLifetime = true,
            ClockSkew = TimeSpan.Zero
        };
    });
```

C'est ici dans ce `service` que le `jwt token` va être vérifié.

On voit que le `service` doit valider la signature : `ValidateIssuerSigningKey = true`. Pour ce faire le `service` a besoin uniquement de la clé secrète : `IssuerSigningKey = new SymmetricSecurityKey(secretKey)`.

Le `service` doit aussi vérifier le temps de validité du `token` avec `ValidateLifetime = true`.

> ## Ajout implicite des `middleware`
>
> Il semble que `UseAuthentication` et `UseAuthorisation` soit ajouter automatiquemnt si les services associés sont ajoutés.
>
> Maintenant pour plus de clarté, rien n'empêche de les ajouter dans `Program.cs`:
>
> ```cs
> app.UseAuthentication();
> app.UseAuthorization();
> ```
>
> 