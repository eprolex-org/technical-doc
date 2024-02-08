# 04 Création en `.net` d'un token `JWT`



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
        access_token = CreateToken(claims, expireAt, configuration),
        expire_at = expireAt
    });
});
```

> Pour utiliser `Results.Forbid()` il faut que le `middleware` d'authorisation soit en place sinon une `exception` est lancée.
>
> ```
> System.InvalidOperationException: Endpoint HTTP: GET /weatherforecast contains authorization metadata, but a middleware was not found that supports authorization.
> ```
>
> Il faut savoir que juste le fait d'ajouter le `service` : `builder.Services.AddAuthorization();` va faire qu'au moment du `build` le `middleware` d'`authorization` sera ajouter automatiquement.

## Création du `access_token`

Ajout des packages `System.IdentityModel.Tokens.Jwt` et `Microsoft.AspNetCore.Authentication.JwtBearer`

```bash
dotnet add package System.IdentityModel.Tokens.Jwt

dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```



```cs
private static string CreateToken(
    IEnumerable<Claim> claims, 
    DateTime expireAt, 
    IConfiguration configuration
)
{
    // On transforme la secret key en tableau de byte
    var secretKey = Encoding.UTF8.GetBytes(configuration["SecretKey"] ?? "");

    // On crée le token
    var jwt = new JwtSecurityToken(
        claims: claims,
        notBefore: DateTime.UtcNow,
        expires: expireAt,
        signingCredentials: new SigningCredentials(
            new SymmetricSecurityKey(secretKey),
            SecurityAlgorithms.HmacSha256Signature
        )
    );

    // ON le serialize en string
    return new JwtSecurityTokenHandler().WriteToken(jwt);
}
```

La `"SecretKey"` se trouve dans `appsettings` :

```json
{
  "Logging": {
    //...
  },
  "SecretKey": "my secret key well secret in configuration"
}
```



### Requête

```http
POST http://localhost:5086/auth
Accept: */*
Content-Type: application/json

{
  "userName": "Admin",
  "password": "password",
  "department": "string"
}
```

### Réponse

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Thu, 08 Feb 2024 13:50:35 GMT
Server: Kestrel
Transfer-Encoding: chunked

{
  "access_token": "eyJhbGciOiJodHRwOi8vd3d3LnczLm9yZy8yMDAxLzA0L3htbGRzaWctbW9yZSNobWFjLXNoYTI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiYWRtaW4iLCJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9lbWFpbGFkZHJlc3MiOiJhZG1pbkBnbWFpbC5jb20iLCJTdXBlciI6InRydWUiLCJEZXBhcnRtZW50Ijoic3RyaW5nIiwiRW1wbG95bWVudERhdGUiOiIyMDIzLTExLTA1IiwibmJmIjoxNzA3NDAwMjM1LCJleHAiOjE3MDc0MDA1MzV9.ffCFh6ymktHQ_lSeTCv2CgNNvejpeueoMBmELHJpvmg",
  "expire_at": "2024-02-08T13:55:35.564463Z"
}
```

