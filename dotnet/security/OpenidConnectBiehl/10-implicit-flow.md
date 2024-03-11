# 10 `Implicit Flow`

Ce flow n'utilise pas le `Back Channel` ni le `Token Endpoint`.

Il est adapté pour les application n'ayant pas de `backend` et tournant dans le navigateur.

Les `Id Token` et `Access Token` sont délivrés par le `Authorization Endpoint`.



## `response_type=id_token token`

Ce `flow` délivre à la fois un `Id Token` et un `Access Token`.

```http
GET /authorize
?response_type=id_token%20token
&scope=openid%20profile%20email
&client_id=p9QwXRkY9p
&state=jsuh83jw72
&nonce=bIUnc89qnc
&redirect_uri=https%3A%F2%F2client/example.org%F2cb 
  HTTP/1.1
host: server.example.com
```









