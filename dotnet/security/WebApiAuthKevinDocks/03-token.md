# 03 Authentification par `token`

<img src="assets/token-flow-tgfde.png" alt="token-flow-tgfde" />

## `J`ason `W`eb `T`oken

Dans le cas de l'authentification d'un `client` par une `api`, `jwt` est un `token` couramment utilisé. 

Un `JWT` se compose de trois parties encodées en `Base64Url` et séparées par des points `.`.



### Header

<img src="assets/jwt-header-on-pluralssight-pobbvesk.png" alt="jwt-header-on-pluralssight-pobbvesk" />

Principalement l'algorithme utilisé pour signer le `jwt`.

<img src="assets/heeader-content-alg-typee-jwt-uuhhyy.png" alt="heeader-content-alg-typee-jwt-uuhhyy" />



### Payload

<img src="assets/jwt-structure-three-part-opljqqa.png" alt="jwt-structure-three-part-opljqqa" />

Le `payload` décodé est en `json` :

<img src="assets/jwt-payload-token-info-user-ttgbbvc.png" alt="jwt-payload-token-info-user-ttgbbvc" />

Ce sont principalement des infos sur le `user`. Ces sont les `claims`.



### Signature

<img src="assets/hash-signature-jwt-oolkgfgfg.png" alt="hash-signature-jwt-oolkgfgfg" />

La `signature` est un `hash` du `payload`.

> ## Définition d'un `hash` (ChatGPT)
>
> Un **hash** est le **résultat d’une fonction de hachage**, qui est un algorithme transformant une entrée (comme une chaîne de caractères, un fichier ou un bloc de données) en une **valeur fixe de longueur déterminée**.
>
> 👉 **Caractéristiques d’une fonction de hachage** :
>
> 1. **Déterministe** : Une même entrée produira toujours le même hash.
> 2. **Rapide** : Calculable efficacement.
> 3. **Unidirectionnel** : Impossible (ou très difficile) de retrouver l’entrée originale depuis le hash.
> 4. **Résistant aux collisions** : Deux entrées différentes ne doivent pas produire le même hash.
> 5. **Légèrement sensible** : Un petit changement dans l’entrée entraîne un changement drastique dans le hash (effet avalanche).
>
> ------
>
> ### 🔹 **Exemples d’algorithmes de hash courants** :
>
> - **MD5** (obsolète pour la sécurité)
> - **SHA-1** (obsolète pour la sécurité)
> - **SHA-256, SHA-512** (utilisés pour la signature JWT, blockchain, etc.)
> - **Bcrypt, Argon2** (utilisés pour les mots de passe, car plus sécurisés contre les attaques par force brute)
>
> ```cs
> string data = "Hello, world!";
> 
> using SHA256 sha256 = SHA256.Create();
> byte[] hashBytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(data));
> string hash = BitConverter.ToString(hashBytes).Replace("-", "").ToLower();
> 
> Console.WriteLine(hash);
> ```
>
> ```
> 315f5bdb76d078c43b8ac0064e4a0164612b1fce77c869345bfc94c75894edd3
> ```
>
> Ce code génère un `hash` de `64` caractères.



## Avantages du `token`

Il offre plus de possibilités de validation (`js front`, `blazor wasm`, ...).

La `signature` prévient de la falsification du `payload`.

La durée de vie est beaucoup plus courte que celle d'une `api key`, le `token` contient un `expiration time` : `exp`.

Transporte des informations sur l'`utilisateur`.



## Implémentation

### `appsettings.json`

```json
// ...
"Authentication" :{
    "SecretForKey": "hysuie786hgfd43ert90lm7gfcx324AErtsfgYUjhgfTY654gt66098ioksltr32z",
    "Issuer": "http://localhost:5005",
    "Audience": "resourceapi"
  }
}
```

`Issuer` est l'entité qui créé le `token`. Notre `API`.

`Audience` c'est l'entité pour laquelle le `token` donne accès.

`SecretForKey` va servir à la génération de la `signature` du `token`.

> ### ! La chaîne `secretForKey` doit au final proposé minimum `256 bytes` pour l'algorithme de `hashage` :
>
> ```
> System.ArgumentOutOfRangeException: IDX10720: Unable to create KeyedHashAlgorithm for algorithm 'HS256', the key size must be greater than: '256' bits, key has '240' bits. (Parameter 'keyBytes')
> ```

## `account endpoint`

```cs
app.MapPost("/account", (Credential credential, IConfiguration configuration) =>
{
    // Vérifier les credentials (via une DB par exemple)
    if (credential is { Name: "hukar", Password: "xxx" })
    {
        // Mon implementation
    }

    return Unauthorized();
});
```

Ajouter le `nuget` :

```bash
System.IdentityModel.Tokens.Jwt
```



### Créer une clé symétrique cryptographique : 

###  `new SymmetricSecurityKey(byte[])`

```cs
// Récupérer la secretKey dans le fichier appsettings
var secret = configuration["Authentication:SecretForKey"] ?? "";

// Convertir laa clé en byte[]
var secretBytes = Encoding.UTF8.GetBytes(secret);

// créer une clé de sécurité
var securityKey = new SymmetricSecurityKey(secretBytes); 
```

> Si la clé secrète est plutôt encodée en `Base64`, on utilise :
> ```cs
> var secretBytes = Convert.FromBase64String(secret);
> ```

C'est cette `securityKey` qui pourra être utilisé par l'algorithme de signature de `jwt`.



### Création de la signature :

###  `new SigningCredentials(securitykey, alg)`

```cs
var signingCredentials = new SigningCredentials(
    securityKey,
    SecurityAlgorithms.HmacSha256
);
```



### Création du `jwt` : `new JwtSecurityToken()`

```cs
public JwtSecurityToken(
    string issuer,
    string audience, 
    IEnumerable<Claim> claims, 
    DateTime? notBefore, 
    DateTime? expires, 
    SigningCredentials signingCredentials
)
```

```cs
// création d'une liste de données représentant le user connecté
List<Claim> claimsForToken =
[
    new("sub", "1234"), // user id
    new("givenname", "hukar"),
    new("email", "hukar@red.be"),
    new(ClaimTypes.Role, "dev"),
];

var securityToken = new JwtSecurityToken(
    configuration["Authentication:Issuer"],
    configuration["Authentication:Audience"],
    claimsForToken,
    DateTime.Now, // begin validity notBefore
    DateTime.Now.AddHours(1), // finish validity Expires
    signingCredentials
);

var tokenString = new JwtSecurityTokenHandler()
    .WriteToken(securityToken);
```

La clé d'une `Claim` peux suivre les conventions ou non. La classe statique `ClaimsType` contient toutes les clés conventionnelles.

> Les `Claims` sont le `payload` du `jwt`, elles représentes les informations de l'utilisateur.
>
> Il est aussi possible d'utiliser `SecurityTokenDescriptor` à la place de `JwtSecurityToken` :
> ```cs
> var key = Encoding.UTF8.GetBytes("ma_super_cle_secrete");
> var credentials = new SigningCredentials(new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256);
> 
> var tokenDescriptor = new SecurityTokenDescriptor
> {
>     Subject = new ClaimsIdentity(new[] { new Claim(ClaimTypes.Name, "admin") }),
>     Expires = DateTime.UtcNow.AddHours(1),
>     SigningCredentials = credentials,
>     Issuer = "monApp",
>     Audience = "monPublic"
> };
> 
> var tokenHandler = new JwtSecurityTokenHandler();
> var securityToken = tokenHandler.CreateToken(tokenDescriptor);
> var tokenString = tokenHandler.WriteToken(securityToken);
> 
> Console.WriteLine(tokenString);
> ```

Résultat du `endpoint` :

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0IiwiZ2l2ZW5uYW1lIjoiaHVrYXIiLCJlbWFpbCI6Imh1a2FyQHJlZC5iZSIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6ImRldiIsIm5iZiI6MTc0MzY4NjIyMCwiZXhwIjoxNzQzNjg5ODIwLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjUwMDUiLCJhdWQiOiJyZXNvdXJjZWFwaSJ9.fGM4YCpre2ajP2SMpPohphn5jG8Ix_K_D-b8hmOhSOs
```

On peut lire le token grâce au site https://jwt.io/

<img src="assets/browser-website-jwt-official-gtqlbdf.png" alt="browser-website-jwt-official-gtqlbdf" />



## sécuriser un `endpoint`

On doit ajouter le `nuget` :

```bash
Microsoft.AspNetCore.Authentication.JwtBearer
```



### Dans `Program.cs`

On va configurer la validation du `token` :

```cs
string secretBytes = Encoding.Utf8.GetBytes(
    builder.Configuration["Authentication:secretForKey"]
) ?? "";

builder.Services.AddAuthentication("Bearer")
    .addJwtBearer(options => {
        options.TokenValidationParmeters = new() {
            // ValidateIssuer = true, par défaut
            // ValidateAudience = true, par dféuat
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Authentication:Issuer"],
            ValidAudience = builder.Configuration["Authentication:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(secretBytes)
        };
    }
);

builder.Services.AddAuthorization(); // obligatoire

// ...

app.UseAuthentication();
app.UseAuthorization();
```

La date d'expiration du `token` est automatiquement validée (valeurs par défaut) :

```cs
public TokenValidationParameters()
{
    LogTokenId = true;
    LogValidationExceptions = true;
    RequireExpirationTime = true;
    RequireSignedTokens = true;
    RequireAudience = true;
    SaveSigninToken = false;
    TryAllDecryptionKeys = true;
    TryAllIssuerSigningKeys = true;
    ValidateActor = false;
    ValidateAudience = true;
    ValidateIssuer = true;
    ValidateIssuerSigningKey = false;
    ValidateLifetime = true; // <= ici
    ValidateTokenReplay = false;
}
```





### Dans le `endpoint`

```cs
var resourceGroup = app
            .MapGroup("/resource")
            .RequireAuthorization(); // <= ici
```



## Test avec un fichier `.http`

<img src="assets/test-authorized-endpoint-http-gstredqll.png" alt="test-authorized-endpoint-http-gstredqll" />

La `resource` est protégée par un `jwt`.

<img src="assets/get-account-token-yeah-jquieoknbcfd.png" alt="get-account-token-yeah-jquieoknbcfd" />

On demande un `jwt` à l'`endpoint account`, on reçoit une chaîne de caractères en `Base64` représentant le `jwt`.

<img src="assets/call-with-jwt-to-get-resource-ggtaajqsmj.png" alt="call-with-jwt-to-get-resource-ggtaajqsmj" />

On ajoute un `Header` : `Authorization` à sa requête avec comme valeur `Bearer <JWT Base64>`.









