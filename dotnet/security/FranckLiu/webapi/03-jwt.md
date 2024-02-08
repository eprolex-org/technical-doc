# 03 Jason Web Token `JWT`

Un `token` est un `string` qui peut être transporté par la `requête` et par la `réponse`.



## Anatomie de `JWT`

`JWT` est un `token` avec trois parties séparées par deux points.

1. l'algorithme de hachage 
2. Les `claims` contenant les informations sur l'utilisateur au format `json`
3. Les `claims` hachées par l'algorithme en guise de signature

Les `claims` sont hachées grâce à l'algorithme et une `clé secrète` et donne le `hashed claims` qui sert de signature. C'est un processus unidirectionnel, on ne peut pas l'inverser.

<img src="assets/jwt-single-way-process.png" alt="jwt-single-way-process" style="zoom:50%;" />

### ! ne pas mettre d'information sensible dans les `Claims`car elles ne sont pas cryptées dans le `JWT`.



## Vérification du `JWT`

Seule les application partageant la même `clé` peuvent vérifier l'authenticité du `JWT` en répétant le processus de génération de la `signature` et en comparant celle-ci à celle du `Token`.

<img src="assets/comparison-token-egal-similitude.png" alt="comparison-token-egal-similitude" style="zoom:33%;" />



Si un `Hacker` essaye de modifier les `Claims`, il devra créer la signature avec sa propre `clé` et celle-ci ne correspondra plus lors de la vérification.

<img src="assets/hacker-key-usurpation.png" alt="hacker-key-usurpation" style="zoom:33%;" />



## `JWT` flow

C'est le `flow` standard où les `Web API` demande la vérification du `token` au `Auth Provider`.

On peut garder l'utilisation des `Cookies` entre l'utilisateur et la `Web App`.

<img src="assets/flow-token-jwt-auth-provider.png" alt="flow-token-jwt-auth-provider" style="zoom:33%;" />

Si les `Web API` et le `Auth Provider` partage la même `clé`, il n'est pas nécessaire de passer par le `Auth Provider` quand la `Web App` demande une ressource à une `Web API`.

<img src="assets/same-key-authorization.png" alt="same-key-authorization" style="zoom:33%;" />





























