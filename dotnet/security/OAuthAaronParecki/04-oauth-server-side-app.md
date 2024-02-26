# 04 `OAuth` pour les `applications` côté serveur (`server side`)



 ## Enregister son `app`

On doit enregistrer son `app` auprès d'un serveur `OAuth`, on doit lui fournir une `URL` de redirection.

On donne quelque information sur son `app` et le `OAuth server` nous retourne un `Client ID` et si c'est un `Confidential Client` un `Client sercret`.

Ces données sont nécessaire pour les afficher aux `utilisateurs` ou pour renseigner les `logs`.

On doit au moins enregistrer le `name` de l'`app` et une ou plusieurs `URL` de redirection.

Cet (ces) `URL(s)` de redirection empêche un attaquant d'utiliser votre `Client ID`, car il ne peut venir de la même `URL`.

Cette `URL` ne doit pas contenir de `wildcard` pour éviter des attaques.

l'`URL` est vraiment le seul moyen pour l'`authorization server` de savoir que ce n'est pas un attaquant qui essaye d'usurper l'identité de l'`app` réel.

L'`app` peut passer son `Client ID` de façon visible au `OAuth server`.

Une fois l'`app` enregistrer, elle est prête à commencer un `OAuth Flow`.

Le `Client ID` est considéré comme donnée public, il peut être visible dans le code source.

L'`Authorization server` ne doit pas renvoyer un `Client Secret` (`password`) pour une `application mobile` ou une `SPA`, si même il le renvoyait, il ne faudrait pas l'utiliser (car il n'y a aucun moyen de le rendre véritablement secret).

Dans le cas d'une `application serveur`, le `client secret` peut être gardé dans le code de l'`application` car ce code n'est pas lu (accessible) par l'`utilisateur`.



## `Authorization Code` Flow

Le but est que l'application `Client` reçoive un `Access Token` du `OAuth Server` via le `Back Channel`, ainsi le navigateur n'a jamais connaissance du `Access Token`.

L'`Authorization Code` a une durée de vie courte, typiquement environ une minute.

La spécification `OAuth` recommande `PKCE` pour tous les types d'applications, même les `applications Serveurs` qui peuvent garder un `Client Secret`.

Il existe une attaque : `Authorization Code Injection Attack`, `PKCE` résoud cette attaque et est maintenant conseillé pour un `Confidential Client` de même que c'était le cas pour un `Public Client`.



### étapes

- Le `Client` génère un `Code Verifier` (`secret`) qui est un `random string` de 43 à 128 caractères de long :
  > [Norme Officielle de PKCE](https://www.rfc-editor.org/rfc/rfc7636.html#page-8)
  >
  > ## Le client crée un `Code Verifier`
  >
  >    Le client crée d'abord un `Code Verifier`, "code_verifier", pour chaque demande d'autorisation OAuth 2.0 [RFC6749], de la manière suivante
  >    OAuth 2.0 [RFC6749], de la manière suivante :
  >
  >    `code_verifier` = `string` aléatoire cryptographique à haute entropie utilisant les
  >    caractères non réservés `[A-Z] / [a-z] / [0-9] / "-" / "." / "_" / "~". / "_" / "~"`
  >    de la section 2.3 de [RFC3986], avec une longueur minimale de `43` caractères
  >    et une longueur maximale de `128` caractères.
  >
  > ...
  >
  >    NOTE : Le `Code Verifier` DEVRAIT avoir suffisamment d'entropie pour qu'il soit impossible de deviner la valeur.
  >    pour qu'il soit impossible de deviner la valeur.  Il est RECOMMANDÉ d'utiliser la sortie d'un générateur de nombres aléatoires approprié pour deviner la valeur.
  >    d'un générateur de nombres aléatoires approprié pour créer une séquence de `32 octets`.  La séquence d'octets est ensuite encodée en `base64url` pour produire une `URL safe string` de `43` octets.
  
  Un caractère en `Base64` est codé sur `6 bits`, c'est pourquoi il suffit de `32 bits` pour générer `43 caractères`.
  Voici une implémentation en `.net` :
  
  ```cs
  string GenerateCodeVerifier()
  {
      var bytes = new byte[32];
      using var rng = RandomNumberGenerator.Create();
      
      rng.GetBytes(bytes); // remplie le tableau
          
      return Base64UrlEncode(bytes);
  }
  
  string Base64UrlEncode(byte[] bytes) => Convert.ToBase64String(bytes)
          .Replace('+', '-')
          .Replace('/', '_')
          .TrimEnd('=');
  ```
  
  `3GQBxy87ZnS25juVOAGmxqAt3xmvWD-0Sg5nefVq-4c`
  
- Le `Client` calcule le `Code Challenge` :
  `code_challenge = BASE64URL-ENCODE(SHA256(ASCII(code_verifier)))`

  ```cs
  string GenerateCodeChallenge(string codeVerifier)
  {
      // Convertir le code verifier en bytes ASCII
      byte[] codeVerifierBytes = Encoding.ASCII.GetBytes(codeVerifier);
  
      // Calculer le hachage SHA-256 du code verifier
      using SHA256 sha256 = SHA256.Create();
  
      byte[] hashBytes = sha256.ComputeHash(codeVerifierBytes);
  
      // Convertir le hachage en Base64URL
      return Base64UrlEncode(hashBytes);
  }
  ```

  `vxyBYF3NSDD3SugkaiBdYoN2BdWxzR9mhT18PJ3HVpw`
  Il n'est normalement pas possible de trouver le `code verifier` en partant du `code challenge`.

- On envoie une requête au `OAuth Server` sur le `Front Channel` avec des `Queries Parameters`
  ```http
  https://auth-server.com/auth?
  	response_type=code&
  	client_id=MY_CLIENT_ID&
  	redirect_uri=MY_REDIRECT_URI&
  	scope=photos&
  	state=xxx&
  	code_challenge=xxxxxxxxxxxx&
  	code_challenge_method=s256
  ```

  Avec `PKCE` ,`state` n'est plus forcement nécessaire, il peut contenir des méta-data comme le numéro de page souhaité. Si on n'utilise pas `PKCE`, il devient obligatoire d'utiliser un `string` aléatoire.

  `s256` = `SHA 256`

- Ensuite le `User` se loggue et il consent à ce qu'il délégue en autorisation, puis le `OAuth Server` renvoie une `URL` de redirection vers le navigateur :
  ```http
  https://my-app.be/redirect?
  	code=AUTH_CODE_HERE&
  	state=xxxxx&
  ```

- Le `Client` check la valeur de `state` pour éviter `CSRF` et créé une requête `POST` en `Back Channel` vers le `endpoint /token`. les données sont transmise grâce à `x-www-form-urlencoded`
  ```http
  POST https://auth-server.com/token
  
  	grant_type=authorization_code&
  	code=AUTH_CODE_HERE&
  	redirect_uri=MY_REDIRECT_URI&
  	code_verfier=VERFIER_STRING&
  	client_id=MY_CLIENT_ID&
  	client_secret=M_CLIENT_SECRET
  ```

  Certain serveur accepte ces données dans le `Header` et d'autre dans le `Body`.

- SI tout fonctionne le `OAuth Server` répond en `json` :
  ```json
  {
      "token_type": "Bearer",
      "access_token": "7ZnS25juVOAG",
      "expires_in": 3600,
      "scope": "photos",
      "refresh_token": "mxqAt3xmvWD"
  }
  ```

- >### Si c'est pour un `refresh_token`
  >
  >On est toujours `Back Channel`.
  >
  >La requête du `Client` est légèrement différente :
  >```http
  >POST https://auth-server.com/token
  >
  >	grant_type=refresh_token&
  >	refresh_token=REFRESH_TOKEN&
  >	client_id=MY_CLIENT_ID&
  >	client_secret=M_CLIENT_SECRET
  >```
  >
  >Si le `OAuth Server` ne reconnait pas le `refresh_token` comme valide, le cycle recommence du début.

