# 20 Double rendu au démarrage

Avec `Blazor server` le composant sont deux fois rendu au démarrage de l'application.

C'est parce que les composant sont pré-rendu, on peut désactiver ce comportement comme ceci :

```xml
<head>
    
<HeadOutlet @rendermode="new InteractiveServerRenderMode(prerender: false)"/>
</head>

<body>

<Routes @rendermode="new InteractiveServerRenderMode(prerender: false)"/>
```



# Documentation officielle (traduction Copilot)


## Rendu anticipé (*Prerendering*)

Le rendu anticipé est le processus de rendu initial du contenu de la page côté serveur **sans activer les gestionnaires d’événements** pour les contrôles rendus. Le serveur génère l’interface HTML de la page dès que possible en réponse à la requête initiale, ce qui rend l’application plus réactive pour les utilisateurs. Le rendu anticipé peut également améliorer le référencement (SEO) en rendant du contenu dans la réponse HTTP initiale, utilisé par les moteurs de recherche pour calculer le classement des pages.

Le rendu anticipé est activé par défaut pour les composants interactifs.

La navigation interne dans le cadre du routage interactif ne nécessite pas de nouvelle requête de contenu de page au serveur. Par conséquent, le rendu anticipé **ne se produit pas** pour les requêtes internes de page, y compris pour la navigation améliorée. Pour plus d’informations, voir :

- *Static versus interactive routing*
- *Interactive routing and prerendering*
- *Enhanced navigation and form handling*

---

## Désactivation du rendu anticipé

La désactivation du rendu anticipé via les techniques suivantes ne prend effet **que pour les modes de rendu de niveau supérieur**. Si un composant parent spécifie un mode de rendu, les paramètres de rendu anticipé de ses enfants sont ignorés. Ce comportement est en cours d’examen pour une éventuelle modification avec la sortie de .NET 10 en novembre 2025.

### Pour désactiver le rendu anticipé d’une instance de composant :

```xml
<... @rendermode="new InteractiveServerRenderMode(prerender: false)" />
<... @rendermode="new InteractiveWebAssemblyRenderMode(prerender: false)" />
<... @rendermode="new InteractiveAutoRenderMode(prerender: false)" />
```

### Pour désactiver le rendu anticipé dans la définition d’un composant :

```ruby
@rendermode @(new InteractiveServerRenderMode(prerender: false))
@rendermode @(new InteractiveWebAssemblyRenderMode(prerender: false))
@rendermode @(new InteractiveAutoRenderMode(prerender: false))
```

### Pour désactiver le rendu anticipé pour toute l’application :

Indiquez le mode de rendu au niveau **du composant interactif le plus haut** dans la hiérarchie de composants de l’application, qui n’est pas un composant racine.

Dans les applications basées sur le modèle de projet Blazor Web App, un mode de rendu assigné à l’ensemble de l’application est spécifié là où le composant `Routes` est utilisé dans le composant `App` (`Components/App.razor`). L’exemple suivant définit le mode de rendu de l’application sur **Interactive Server** avec le rendu anticipé désactivé :

```xml
<Routes @rendermode="new InteractiveServerRenderMode(prerender: false)" />
```

Désactivez également le rendu anticipé pour le composant `HeadOutlet` dans le composant `App` :

```xml
<HeadOutlet @rendermode="new InteractiveServerRenderMode(prerender: false)" />
```

> ❌ Il n’est **pas pris en charge** de rendre un composant racine, tel que le composant `App`, interactif via la directive `@rendermode` placée en haut du fichier de définition du composant (.razor). Par conséquent, le rendu anticipé **ne peut pas être désactivé directement** par le composant `App`.
