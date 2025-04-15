# 02 le middleware `OpenIdConnect`

```bash
dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect
```



## Configuration du `middleware`

```ruby
const string entraIdScheme = "EntreIdOpenIdConnect";

builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = entraIdScheme;
}).AddOpenIdConnect(entraIdScheme, oidcOptions =>
{
    oidcOptions.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    oidcOptions.Authority = "https://login.microsoftonline.com/91acc1c2-22b4-4555-b75e-7fc1f9188a09/v2.0"; // issuer
    oidcOptions.ResponseType = OpenIdConnectResponseType.Code; // code flow
    oidcOptions.UsePkce = true;
    oidcOptions.Scope.Add(OpenIdConnectScope.Profile);
    oidcOptions.CallbackPath = new PathString("/signin-oidc"); // defaukt value
    oidcOptions.SignedOutCallbackPath = new PathString("/signout-callback-oidc"); // default value
    oidcOptions.ClientId = "e9103ed1-540d-4d4e-b23e-0fc63c9514a6";
    oidcOptions.ClientSecret = "~5T8Q~B0~VumB~i8C5jtCBYEJY.LfHWijCYmsa42";
    oidcOptions.MapInboundClaims = false;
    oidcOptions.TokenValidationParameters.NameClaimType = JwtRegisteredClaimNames.Name;
    oidcOptions.TokenValidationParameters.RoleClaimType = "Role";
    oidcOptions.SaveTokens = true;

}).AddCookie(CookieAuthenticationDefaults.AuthenticationScheme);
```

> ChatGPT:
>
> #### 🧩 Les options internes :
>
> | Ligne                                                        | Explication                                                  |
> | ------------------------------------------------------------ | ------------------------------------------------------------ |
> | `oidcOptions.SignInScheme = "Cookies";`                      | Une fois l'utilisateur connecté via OIDC, le résultat est stocké dans un **cookie d'authentification**. |
> | `oidcOptions.Authority = "https://login.microsoftonline.com/{tenant-id}/v2.0";` | C’est le **serveur d’autorité** (Azure Entra ID). Le middleware va interroger `/.well-known/openid-configuration`. |
> | `oidcOptions.ResponseType = "code";`                         | On utilise le **Code Flow** (le plus sécurisé, recommandé).  |
> | `oidcOptions.UsePkce = true;`                                | Active le **PKCE** pour renforcer la sécurité du code flow (même en server-side, c’est de plus en plus standard). |
> | `oidcOptions.Scope.Add("profile");`                          | Demande le scope **`profile`** (retourne nom, prénom, etc.). `openid` est automatiquement ajouté. |
> | `oidcOptions.CallbackPath = "/signin-oidc";`                 | URL sur laquelle Azure redirige après l'auth. (par défaut)   |
> | `oidcOptions.SignedOutCallbackPath = "/signout-callback-oidc";` | Redirection après un logout côté Azure (par défaut aussi).   |
> | `oidcOptions.ClientId / ClientSecret`                        | Ton application enregistrée sur Entra ID.                    |
> | `oidcOptions.MapInboundClaims = false;`                      | Par défaut, .NET "remappe" les claims pour les rendre plus .NET-like. Ici on garde les noms bruts. |
> | `TokenValidationParameters.NameClaimType = "name";`          | Indique quel claim représente le nom de l’utilisateur.       |
> | `TokenValidationParameters.RoleClaimType = "Role";`          | Indique le claim à utiliser pour les rôles (attention, Azure n’envoie pas `Role` par défaut). |
> | `SaveTokens = true;`                                         | Sauvegarde les tokens (`id_token`, `access_token`, `refresh_token`) dans le `AuthenticationProperties` → utile si tu veux les réutiliser plus tard. |
>



## `Path`, `PathString` et `Uri` 

(ChatGPT)

### 🔹 `System.IO.Path`

- **But** : Travailler avec **les chemins de fichiers système** (disques, dossiers, fichiers).
- **Exemples** :
  - `Path.Combine("C:\\Users", "MonFichier.txt")`
  - `Path.GetExtension("fichier.txt") → ".txt"`
- **Utilisation typique** : Lecture/écriture de fichiers, manipulation de chemins sur le disque.
- 📁 **Concerne le système de fichiers local**.



### 🔹 `Microsoft.AspNetCore.Http.PathString`

- **But** : Représenter et manipuler **les segments d’URL dans le contexte d’une requête HTTP** (surtout dans ASP.NET Core).
- **Exemples** :
  - `var pathString = new PathString("/api/products")`
  - `pathString.Add("/details") -> "/api/products/details"`
  - Utilisé dans des middlewares, pour router des requêtes, etc.
- **Avantages** :
  - Encapsule les chemins d’URL (avec gestion d’encodage, comparaison sécurisée, etc.)
- 🌐 **Concerne le chemin de la requête HTTP**, sans protocole ni domaine.



### 🔹 `System.Uri`

- **But** : Représenter **une URL complète** (ou URI générique, comme une base de données, FTP, etc.)

- **Exemples** :

  - `new Uri("https://example.com/api/products?id=1")`
  - `uri.Host → "example.com"`
  - `uri.AbsolutePath → "/api/products"`

- **Fonctionnalités** :

  - Analyse complète de l’URI (schéma, port, query string, fragment…)

- 🌍 **Concerne des adresses complètes (web ou autre)**.

  

------

### 🧠 En résumé

| Classe       | Domaine         | Représente                          | Exemples d'usage                     |
| ------------ | --------------- | ----------------------------------- | ------------------------------------ |
| `Path`       | Fichiers locaux | Chemin de fichiers sur disque       | `C:\Dossier\Fichier.txt`             |
| `PathString` | ASP.NET Core    | Partie *chemin* d’une URL HTTP      | `/api/products`                      |
| `Uri`        | Global / Web    | URI complet (protocole, hôte, etc.) | `https://site.com/api/products?id=1` |



## Utilisation de `appsettings.json`

Le `middleware` d'authentification va chercher automatiquement ses valeurs dans `appsettings` :

```json
"Authentication": {
    "Schemes": {
      "EntraIDOpenIDConnect": {
        "Authority": "https://login.microsoftonline.com/5c154a7e-0c13-4f92-8531-e3f4d8fbeae9/v2.0",
        "ClientId": "3b2f51e7-ea7d-4a0f-931f-87e98e6b022e",
        "ClientSecret": "",
        "CallbackPath": "/signin-oidc",
        "SignedOutCallbackPath": "/signout-callback-oidc",
        "ResponseType": "code"
      }
    }
  }
```

