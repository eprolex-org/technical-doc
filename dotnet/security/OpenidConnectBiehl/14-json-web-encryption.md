# 14. `JWE` : `Json Web Encryption`

Contenu crypté en utilisant une data structure basé sur `Json`.

Le contenu n'est pas forcement du `Json`, cela peut-être un `string` (par exemple un `JWS`).

Peut-être sérialisé de deux manières :

- `JWE Compact Serialization`
  - compacte et. c'est une représentation `URL - Safe`
  - Pour les `header` d'`Authorization` `HTTP`
  - Pour les `Query Parameters` des `URI`
  - Peut seulement inclure le `JWE Protected Header` pas le `JWE Unprotected Header`
- `JWE Json Serialization`
  - Représente le `JWE` en `Json`



## Contenu d'un `JWE`

- `Header` (`JOSE Header`)
  - `Protected Header`
  - `Shared Unprotected Header`
  - `Per-Recipient Unprotected Header`
- `Encrypted Key`
- `Initialization Vector`
- `Ciphertext`
- `Additional Authenticated Data`
- `Authentication Tag`

​		

## Champs du Header

Ce ne sont pas des `claims` mais des attributs.

<img src="assets/all-header-jwe-michel.png" alt="all-header-jwe-michel" />

Ce sont les mêmes que `JWS` plus deux nouveaux

### `enc`

Nom de l'algorithme de cryptage du contenu.Les valeurs possibles sont définies par `JWA` (`Json Web Algorithm`) registre.

### `zip`

nom de l'algorithme de compression, par exemple `DEF` pour `deflate algorithme`, qui est appliqué sur le `plaintext` avant d'être crypté.

Il y a deux types d'algorithme de cryptage dans un `JWE` :

- `key encryption algorithme` : `alg`
- `content encryption algorithme` : `enc`



## `JWE` Sub-Header

Les champs de `Header` peuvent être séparés en trois :

- Protected Header
- Shared Unprotected Header
- Per-Recipient Unprotected Header

`Protected Header` font partis de `JWE Encryption Input`

`Shared` et `Per-Recipient` `Unprotected Header` ne font pas partis de `JWE Encryption Input`.

`JWE Compact serialization` ne supporte que `Protected Header`.

Si on a des `Unprotected Header` on doit utiliser plutôt `Json serialization`.