# 19 Nouveautés `.Net 9`



## Gestion des fichiers statique

Une meilleur gestion des fichiers statique, avec une meilleur compression est proposé dans `.net 9`.

Il suffit de changer quelques petites choses :

`MapStaticAssets`

```cs
app.UseStaticFiles();

// on le change en ceci

app.MapStaticAssets();
```

Puis dans `App.razor` on encapsule tous les fichiers statiques dans `@Assets["..."]`

```html
<head>
    // ...
    <base href="/"/>
    <link rel="stylesheet" href="app.css"/>
    <link rel="stylesheet" href="BlazorServerUI.styles.css"/>
    <link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" rel="stylesheet"/>
    <link href="_content/MudBlazor/MudBlazor.min.css" rel="stylesheet"/>
```

devient :

```html
<head>
    // ...
    <base href="/"/>
    <link rel="stylesheet" href="@Assets["app.css"]"/>
    <link rel="stylesheet" href="@Assets["MudBlazorResponsiveButton.styles.css"]"/>
    <link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" rel="stylesheet"/>
    <link href="@Assets["_content/MudBlazor/MudBlazor.min.css"]" rel="stylesheet"/>
```

