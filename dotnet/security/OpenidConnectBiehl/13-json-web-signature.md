# 13. `JWS` : `Json Web Signature`

Représente un contenu sécurisé par une signature digitale ou `Message Authentication Codes` (`MACs`) utilisant une structure de donnée basée sur `JSON`.



## Deux `serializations` pour `JWS`

### `JWS` Compact Serialization

-  une serialization compacte et `URL-safe`
- Pensée pour des environnement avec contrainte d'espace
- Peut-être utilisé dans les `HTTP Authorization headers`
- Peut-être utilisé dans des `URI` comme `Query Parameters`
- Ne peux pas représenter le `header` non protégé.



### `JWS` `JSON` serialization

Plus lisible, peut représenter le `header` non protégé.



## Anatomie d'un `JWS`

<img src="assets/anatomy-jws-three-parts-its-ok.png" alt="anatomy-jws-three-parts-its-ok" />

### `JWS` header

`JOSE Header` = `JSON Object Signing and Encryption`

Consiste en :

- `JWS` Protected Header
- `JWS` Unprotected Header (ne peux pas être représenté avec `Compact Serialization`)

Les champs du `header` ne sont pas des `claims` ce sont justes des metadatas permettant de construire le `JWS`.

`alg` : L'algorithme utilisé pour signer le `JWS` (depuis le répertoire `JWA`). Un `JWT` non sécurisé est un `JWS` utilisant comme valeur pour `alg` : `none`.



### `JWS` Payload



### `Signature`

