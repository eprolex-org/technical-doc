# 04. `Tokens` et `Credentials`



## Différents `Token`

### `Access Token`

Utilisé par le `Client` pour accéder aux `ressources`, il a une durée de vie limité (par exemple 24h).

Pendant toute la période de validité, le même `Access Token` peut être réutilisé pour n'importe quel nombre de requête.

C'est un `Bearer token`, le porteur du `token` a ses droits d'accès associés à celui-ci.

Si vous avez le `Token`, peut importe qui vous êtes et comment vous l'avez eu, vous devez juste le présenter, c'est pourquoi il faut garder confidentiel le `Access Token`.



### `Refresh Token`

Le `Refresh Token` a une période de validité supérieur à celle du `Access Token`, par exemple 30 jours ou 3 mois.

Le `Refresh Token` peut être utilisé pour demander un nouvel `Access Token` après que l'`Access Token` soit périmé. Lorsque le `Refresh Token` est utilisé, il n'est pas nécessaire de redemander les `credentials` au `Resource Owner`, il est considéré comme un court-circuit pour `Authorization Code flow`, `Client Credential Flow` et `Resource Owner Password Credential Flow`.

Les `Refresh Token` sont conservé et envoyé par le `Client`.

Il sont envoyé au `/token` `endpoint` du `OAuth Server`.

Le `Refresh Token` n'est jamais envoyé au `Resource Server`, on n'utilise pas le `Rrefresh Token` pour accéder à une `Resource`.



### `Authorization Code`

C'est la confirmation que le `Resource Owner` est authentifié correctement et qu'il a donné son consentement.

L'`Authorization Code` a une durée de validité courte limitée à quelques minutes. Juste assez de temps pour que le `Client` puisse demander un `Access Token` au `Token Endpoint`.

L'`Authorization Code` peut uniquement être utilisé par le `OAuth server`, il n'est jamais envoyé au `Resource Server`.



## Différents `Credentials`

### `Resource Owner Credentials`

Le `Resource Owner` donne ses `Credentials` uniquement à l'`OAuth Server`, jamais au `Client`.

### `Client Credentials`

`ClientId` et `ClientSecret`, utilisés pour accéder au `Token Endpoint` via `Http basic`.

```ruby
Authorization: Basic base64(username:password)
```

> Il existe aussi aussi `OAuth Token Authentication`
>
> ```ruby
> Authorization: Bearer <access_token>
> ```
>
> - **Basic** : Il s'agit d'un schéma d'authentification HTTP standard où les identifiants (nom d'utilisateur et mot de passe) sont envoyés sous forme de texte brut, encodés en Base64, dans l'en-tête Authorization de la requête HTTP. Cette méthode est largement utilisée pour l'authentification de base dans de nombreux systèmes HTTP.
> - **Bearer** : C'est un autre schéma d'authentification HTTP où un jeton d'accès est envoyé dans l'en-tête Authorization de la requête HTTP. Ce jeton d'accès est généralement émis par un serveur d'autorisation lorsqu'un utilisateur s'authentifie avec succès. Le serveur qui reçoit le jeton d'accès peut alors le valider pour accorder l'accès aux ressources protégées associées à ce jeton.
>
> **ChatGPT**

Le `Client` doit s'enregistrer (`Client Registration`) auprès du `OAuth Server` pour recevoir un `ClientId` (et un `ClientSecret`).



### `Access Token`

Comme il doit être considéré comme un `Credential`, il doit être protégé dans ce sens (gardé secret).



### `Refresh Token`

Qui lui aussi doit être stocké de manière sécurisée (car il permet d'obtenir un `Access Token` qui permet d'accéder aux `Resources`).



### `Authorization Code`

De même que `Refresh Token`, il permet d'obtenir un `Access Token`.



## `Client Registration` : enregistrement du `Client`

Le `Client` doit s'enregistrer auprès du `OAuth Server` en fournissant certaine information à propos de lui. Il reçoit en échange un `ClientId` et un `ClientSecret`.

Le `Client` doit fournir :

- `Redirect URI`
- `Scopes` désirés

