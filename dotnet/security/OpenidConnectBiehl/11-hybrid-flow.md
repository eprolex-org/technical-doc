# 11. `Hybrid` Flow

Ce `flow` n'existe pas dans `OAuth` mais seulement dans `OPenId Connect`.

Il y a `3` `Hybrid Flow` différents.



## `Hybrid` Flow                                    `response_type=code id_token token`

<img src="assets/hybrid-flow-first-one-of%20three.png" alt="hybrid-flow-first-one-of three" />

Il reçoit à la fois le `authorization code` s'il veut demander proprement un `Access Token` et s'il souhaite faire un raccourci, directment l'`Id` et l'`Access` `Token`.

```http
GET /authorize
?response_type=code%20id_token%20token
&scope=openid%20profile%20email
&client_id=p9QwXRkY9p
&state=jsuh83jw72
&redirect_uri=https%3A%F2%F2client/example.org%F2cb 
  HTTP/1.1
host: server.example.com
```

Le paramètre `nonce` n'est pas requis dans l'`Hybrid Flow`.

```http
HTTP/1.1 302 Found
Location: https://client.example.org/cb
#code=FghYTR45ju78fdER32sxC
&access_token=aHVrYXI6bXlQQHNzdzByckQ
&token_type=Bearer
&id_token=p9QwXRk.Y9pFghYTR45ju.78fdER32sxC
&state=jsuf56tfsg83
```

C'est aussi un `Fragment` et pas des `Query Parameters`.

L'intérêt d'avoir aussi un `Authorization Code` et de pouvoir récupérer un `refresh_token` sur le `Token Endpoint`.

On peut utiliser le `Back Channel` pour cela.



## Validation pour l'`Hybrrid Flow`

### vérifier l'`Id Token`

Le `claim` : `nonce` est obligatoire dans l'`Id Token`.

`at_hash` et `c_hash` sont obligatoire dans l'`Id Token`.

Si un `Id Token` est retourné à la fois par l'`Authorization Endpoint` et le `Token Endpoint`, la valeur des `claims` : `iss` et `sub` doivent être identique dans les deux `Id Token`. Les autres `claims` devraient aussi être les mêmes.

Le `Client` doit valider la signature de l'`d Token` (`JWS`).

Le `Client` doit vérifier que la valeur de `nonce` est la même que celle envoyée dans l'`Authorization Request`.



### Vérifier que l'`Authorization Code` et l'`Id Token` sont compatibles

Obtenir le `Hash` des octets en représentation `ASCII` de l'`authorization_code` en utilisant le paramètre `alg` de l'en-tête (`header`) de l'`Id Token` (`JOSE Header`). Si `alg` vaut `RS256` alors l'algorithme utilisé est `SHA-256`.

La deuxième moitié la plus à gauche du `hash` est encodée en `Base64Url`.

la `claim` :  `c_hash` (`athorization code hash`) de l'`Id Token` doit correspondre au résultat obtenu précédemment.



### Vérifier que l'`Access Token`  et l'`Id Token` sont compatibles

Obtenir le `Hash` des octets en représentation `ASCII` de l'`access_token`en utilisant le paramètre `alg` de l'en-tête (`header`) de l'`Id Token` (`JOSE Header`). Si `alg` vaut `RS256` alors l'algorithme utilisé est `SHA-256`.

La deuxième moitié la plus à gauche du `hash` est encodée en `Base64Url`.

la `claim` :  `at_hash` (`access token hash`) de l'`Id Token` doit correspondre au résultat obtenu précédemment.



## `Hybrid` Flow `response_type=code id_token`

Ce `flow` permet d'utiliser l'`Id Token` très tôt, d'avoir les info sur le `User` avant d'avoir un `Access Token`.

```http
GET /authorize
?response_type=code%20id_token
&scope=openid%20profile%20email
&client_id=p9QwXRkY9p
&state=jsuh83jw72
&redirect_uri=https%3A%F2%F2client/example.org%F2cb 
  HTTP/1.1
host: server.example.com
```

On obtient la réponse asynchrone par un `Redirect` :

```http
HTTP/1.1 302 Found
Location: https://client.example.org/cb
#code=FghYTR45ju78fdER32sxC
&id_token=p9QwXRk.Y9pFghYTR45ju.78fdER32sxC
&state=jsuf56tfsg83
```

De nouveau on obtient un `Fragment` contenant le `code` et l'`id_token`.



## `Hybrid` Flow `response_type=code token`

On peut utiliser l'`API` en un seul round (un seul appelle) en recevant directement l'`Access Token`.

```http 
GET /authorize
?response_type=code%20token
&scope=openid%20profile%20email
&client_id=p9QwXRkY9p
&state=jsuh83jw72
&redirect_uri=https%3A%F2%F2client/example.org%F2cb 
  HTTP/1.1
host: server.example.com
```

et la réponse asynchrone :

```http
HTTP/1.1 302 Found
Location: https://client.example.org/cb
#code=FghYTR45ju78fdER32sxC
&access_token=aHVrYXI6bXlQQHNzdzByckQ
&token_type=Bearer
&state=jsuf56tfsg83
```

