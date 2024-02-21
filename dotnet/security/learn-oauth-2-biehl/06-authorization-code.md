# Authorization Code `Flow`

## Fonction de ce `Flow`

<img src="assets/auth-code-flow-schema-intro.png" alt="auth-code-flow-schema-intro" style="zoom:150%;" />

Ce `Code Flow` s'appelle aussi `Three Legged Flow` car il vérifie l'identité des trois des acteurs impliqués :

1. `OAuth Server` son identité est vérifié grâce au `certficat` et l'`URL` par le `Client`.
2. `Resource Owner` avec les `Credential` vérifiés par le `OAuth Server`.
3. `Client` avec `ClientId` et `ClientSecret` vérifié par le `OAuth Server`.



## Quand l'utilisé

Quand le `Client` peut fournir un stockage sécurisé du `ClientId`, `ClientSecret` et des `Tokens`. C'est le cas si le `Client` est de type `Server Side Application`.

Si ce stockage sécurisé n'est pas possible, l'utilisation de ce `Flow` n'est pas recommandé.



## Avantage

L'avantage principal ets le haut niveau de sécurité. Par exemple les `Tokens` ne transitent pas par le navigateur et aussi que le `username` et le `password` du `Resource Owner` ne sont jamais connus du `Client`.

C'est aussi sécurisé parceque l'identité de trois des partenaires est assurée.

Ce `Flow` est aussi agréable pour le `Resource Owner` car grâce au `Refresh Token`, il n'a pas besoin de réintroduire ses `credentials` encore et encore.



## `Authorization Code Grant`

- Obetnir l'`Authorization Code`
- Obtenir le `Token`
- Utiliser le `Token` pour accéder à la `Resource`



## `Authorization` Endpoint

C'est le `endpoint` permettant de demander l'`Authorization Code`.

<img src="assets/authorization-schema-endpint-flow-code.png" alt="authorization-schema-endpint-flow-code" />

Requête du `Client` vers le `OAuth Server`  pour demander un `Authorization Code` (`response_type=code`) utilisant les `Queries Parameters`.

```http
GET /authorize
?response_type=code
&scope=name%20phone%20birthdate
&client_id=6s42eRt9
&state=Tg5ds3E
&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb HTTP/1.1

Host: server.example.com
```

`redirect_uri` est `URL Encoded`.

Le `OAuth Server` vérifie que le `client_id` et la `redirect_uri` correspondent bien avec ce qui a été enregistré (`Client Registration`).

Le `OAuth Server` vérifie les `credentials`.

Le consentement se fait sur le `scope` envoyé à l'`OAuth Server`, ici `name`, `phone` et `birthdate`.

Réponse (redirection `302`) du `OAuth Server` vers le navigateur puis redirection vers le `Client`.

```http
HTTP/1.1 302 Found
Location: https://client.example.org/cb
?code=SFD7huIol8
&state=Tg5ds3E
```

On remarque que le `state` est identique.

À la fin de cette séquence, le `Client` est en possession de l'`Authorization Code`.

Le `Resource Server` n'interagit pas dans cette séquence.



## `Token` endpoint

> ## `application/x-www-form-urlencoded` expliqué (ChatGPT)
>
> Lorsque le type de contenu est `application/x-www-form-urlencoded`, les données du formulaire sont encodées dans une chaîne de texte selon certaines règles. En gros, chaque paire clé-valeur est encodée avec le signe égal (`=`) entre la clé et la valeur, et les différentes paires sont séparées par le caractère `&`. De plus, les caractères spéciaux sont encodés en utilisant un encodage spécifique appelé `URL encoding`, où un caractère est remplacé par un pourcentage (`%`) suivi de sa valeur `ASCII` en hexadecimal.
>
> Par exemple, si vous avez un formulaire avec les champs "nom" et "email" et que vous soumettez les données "John Doe" pour le nom et "john@example.com" pour l'email, la chaîne encodée pourrait ressembler à ceci : `nom=John%20Doe&email=john%40example.com`.
>
> - **`x-`**: Cette partie indique qu'il s'agit d'un type de contenu non standard, c'est-à-dire qu'il n'est pas spécifié dans les normes officielles, mais largement utilisé dans la pratique.
> - **`www`**: Cela fait référence au World Wide Web, le système d'information hypermédia qui permet l'accès aux ressources sur Internet. Dans ce contexte, cela signifie que les données sont encodées d'une manière spécifique pour être envoyées via le Web.
> - **`form-urlencoded`**: Cette partie précise le type d'encodage utilisé pour représenter les données. Dans ce cas, les données du formulaire sont encodées en utilisant une méthode spécifique où les caractères spéciaux sont remplacés par des séquences de caractères qui peuvent être transmises via une `URL`. Cela inclut l'utilisation de `%xx` pour les caractères non `ASCII`, ainsi que le remplacement de l'espace par le caractère `+`.

<img src="assets/tokrn-endpoint-schema-auth-grant-flox.png" alt="tokrn-endpoint-schema-auth-grant-flox" />

Le `Client` anvoie une requête `POST` au `OAuth Server` contenant l'`Authorization Code`.

Le contenu est du type `Web Form` : `x-www-form-urlencoded` (voire pus haut).

le `Header` `Authorization` contient le `ClientId` et le `ClientSecret`  encodé en `Base64` et séparé par deux points  `:` 

```ruby
Authorization: Basic base64(username:password)
```

```http
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic czSD45rtYujh76tGbmm0j

grant_type=authorization_code
&code=SFD7huIol8
```

`code` est celui reçu par le `Flow` précédent sur le `/authorize` endpoint.

Le `OAuth Server` va valider cette requête, les `credentials` du `Client`, le `code` et aussi son temps de validité (un temps court de l'ordre de quelques minutes).

Si tout est valide le `OAuth Server` renvoie `200 Ok` et un `json` contenant les `Tokens`:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
	"access_token": "ftYhJl87c5sdSq32jK",
	"token_type": "Bearer",
	"refresh_token": "XxdFtyuhBB95j4",
	"expire_in": 3600
}
```

L'`Access Token` est un `string` qui permettra d'accéder à la `resource` demandée.



## `Resource` endpoint

<img src="assets/obtain-resource-from-data-server-ok.png" alt="obtain-resource-from-data-server-ok" />

Le `Client` envoie sa requête au `Resource Server` en joignant l'`Access Token` obtenu précédemment dans le `header` : `Authorization: Bearer xxx`.

```http
GET /api
Authorization: Bearer ftYhJl87c5sdSq32jK
Host: server.example.com
```

Le `Resource Server` doit maintenant vérifier la validité du `Token` auprès du `OAuth Server`.

Il utilise l'`endpoint` dédié du `OAuth Server` : `/verify`.

Si tout est `Ok`, le `Resource Server` retourne la `Resource` avec un `200 Ok` :

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
	"info": "data"
}
```



## `Refresh Token` flow

<img src="assets/authorization-code-basic-schema-barre-code.png" alt="authorization-code-basic-schema-barre-code" />

Seul le `/token` endpoint est utilisé pour ce `flow`.

Le `Refresh Token` a obligatoirement un temps de validité supérieur à l'`Access Token`, ce temps de validité n'apparaît pas pour le `Client` mais est assigné sur le `OAuth Server`.

`expire_in` dans la réponse du `OAuth Server` est en milliseconde et concerne l'`Access Token`.

Le `Refresh Token` permet au `Client` de sauter l'étape d'authentification du `User` et de directement récupérer un `Access Token` lorsque l'ancien n'est plus valide (périmé).

C'est une sorte de court-circuit de l'`Authorization Code Flow` de départ, posséder le `Refresh Token` signifie que le `Resource Owner` a accepté de partager une `Resource` avec ce `Client` juste avant, pas besoin de lui redemander son accord.

<img src="assets/refresh-tokne-in-valid-period-of-time.png" alt="refresh-tokne-in-valid-period-of-time" />

Dans la période de temps où l'`Access Token` est invalide et le `Refresh Token` est lui toujours valide, on peut envoyé une requête de `Refresh Token` :

```http
POST /token HTTP/1.1
Host: server.example.com
Content-Type: x-www-form-urlencoded
Authorization: Basic czSD45rtYujh76tGbmm0j

grant_type=refresh_token
&refresh_token=XxdFtyuhBB95j4
```

On obtient en retour du `OAuth Server` un nouvel `Access Token` et aussi un nouveau `Refresh Token` avec une nouvelle date d'expiration pour chacun.

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
	"access_token": "Tygh78j3ss2dftlp7",
	"token_type": "Bearer",
	"refresh_token": "vGttUhn6f3d100ol",
	"expire_in": 3600
}
```

<img src="assets/happy-flow-for-refresh-token.png" alt="happy-flow-for-refresh-token" />







