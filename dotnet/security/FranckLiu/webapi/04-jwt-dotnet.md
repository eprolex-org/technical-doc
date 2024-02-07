# 04 Implémentation en `.net` de `JWT`



## Création du `Endpoint`

```cs
app.MapPost("/auth", (Credential credential) =>
{
    if (credential is not { UserName: "admin", Password: "password" }) return Forbid();

    List<Claim> claims = [
        new Claim("UserName", credential.UserName),
        new Claim("Role", credential.Role)
    ];

    var expireAt = DateTime.UtcNow.AddMinutes(10);

    return Ok(new
    {
        access_token = "",
        expire_at = expireAt
    });
});
```



## Création du `access_token`

Ajout des packages `System.IdentityModel.Tokens.Jwt` et `Microsoft.AspNetCore.Authentication.JwtBearer`

```bash
dotnet add package System.IdentityModel.Tokens.Jwt

dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```



```cs
private static string CreateToken(IEnumerable<Claim> claims, DateTime expireAt)
{

    return "";
}
```

