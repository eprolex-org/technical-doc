# 00 `HttpContext`



## Get started

Une application `Blazor Server` partage sa mémoire entre 10,100 ou 1000 utilisateur simultanés.

Chaque utilisateur obtient son propre `circuit` possédant son propre `scope`, son propre conteneur `scoped` pour l'injection de dépendance.

> ## Pour info
>
> https://learn.microsoft.com/en-us/answers/questions/1251249/blazor-server-maximum-concurrent-users-without-sep
>
> ### Nombre d'utilisateurs maximum (réponse Daniel Roth)
>
> Nous avons mis une application `Blazor Server` sous charge avec des clients actifs et avons surveillé la latence des interactions avec les utilisateurs. Dans nos tests, une seule instance `Standard_D1_v2` sur `Azure` (1 vCPU, 3,5 Go de mémoire) pouvait gérer plus de `5 000` utilisateurs simultanés sans aucune dégradation de la latence. Une instance `Standard_D3_V2` (4 vCPU, 14 Go de mémoire) a géré plus de `20 000` clients simultanés.

Dans ce cadre il n'est pas garantie (comme dans une application serveur standard `ASP.NET`) d'avoir accès à `HttpContext` ou bien qu'il soit relié à une `session` particulière.

### En général, on ne doit pas accéder à `HttpContext` dans une application `Blazor Server`



## Accéder au `User`

On utilise `HttpContext` pour connaître quelque chose sur le `user`.

Pour accéder à `HttpContext` on utilise les `cascadingParameter` :

`Home.razor`

```cs
@page "/"

<PageTitle>Home</PageTitle>

<h1>Hello, world!</h1>

Welcome to your new app.

@code {

    [CascadingParameter] public HttpContext? HttpContext { get; set; }

    // ...

}
```

`HttpContext` n'est fiable que lors de la première requête `http`, ensuite ce sont les circuits `Signal R` qui prennent le relais. On va donc créer un service récupérant les infos souhaitées lors de la requête initiale et les mettant à disposition par la suite pour les composants.

> ## Recommandation Doc Microsoft
>
> [HttpContext](https://learn.microsoft.com/fr-fr/dotnet/api/microsoft.aspnetcore.http.httpcontext) peut être utilisé comme [paramètre en cascade](https://learn.microsoft.com/fr-fr/dotnet/api/microsoft.aspnetcore.components.cascadingparameterattribute) uniquement dans les *composants racines rendus statiquement* pour les tâches générales, telles que l’inspection et la modification d’en-têtes ou d’autres propriétés dans le composant `App` (`Components/App.razor`). La valeur est toujours `null` pour le rendu interactif.
>
> ### Remarque:
>
> Dans mes tests `HttpContext` n'est pas `null` dans un composant interactif, mais l'interactivité  est déterminé de manière global.



## Créer un service pour garder les données de `HttpContext`

On sait qu'on peut récupérer au démarage les données d'`HttpContext` dans le composant `root` : `App.razor`, on va créer un service conservant ces données pour le reste de la vie de l'application (avant un éventuel `refresh`).

`IdentityProvider.cs`

```cs 
public class IdentityProvider
{
    public bool IsAuthenticated { get; set; }
    public string Username { get; set; } = string.Empty;
}
```





