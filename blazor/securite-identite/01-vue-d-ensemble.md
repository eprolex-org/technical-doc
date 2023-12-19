# 01 Vue d'ensemble

Résumé de la documentation officielle.

## `Anti Forgery`

Un jeton `AntiforgeryToken` est automatiquement ajouté aux `EditForm` en temps que champ masqué.



## `Authentication`

Elle peut être réalisée avec un `Cookie` ou un `Bearer Token` (jeton porteur).

Elle est gérée dans le `hub SignalR`.



### État partagé

> Les services singletons d'état ne sont pas conseillé avec `Blazor Server`.



## Service `AuthenticationStateProvider`

C'est le service utilisé par le composant `AuthorizeView`.

Ne s'utilise généralement pas directement.