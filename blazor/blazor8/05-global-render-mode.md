# 05 Mode de rendu `global`

## Activer un mode de rendu manuellement

Dans le fichier `App.razor` on a par défaut

```ruby
<head>
    // ...
    <HeadOutlet />
</head>

<body>
    <Routes />
```

Pour activer manuellement un mode de rendu on écrit :

### `@rendermode="@InteractiveServer"`

```ruby
	// ...
	<HeadOutlet @rendermode="@InteractiveServer" />
</head>

<body>
    <Routes @rendermode="@InteractiveServer" />
```

Alors le mode par défaut d'un composant est `InteractiveServer`.

```ruby
<ClickIncrement />
```

le composant est bien interactif et ouvre un `chanel` `SignalR` (`Web Socket`).



### ! Limite 

Il ne semble pas possible de choisir dans ce cas un mode par composant ou par page et ce code génère une `exception` :

```ruby
<ClickIncrement @rendermode="InteractiveWebAssembly" />
```
<span style="color: red;">System.NotSupportedException: Cannot create a component of type 'BlazorSSR.Client.Components.ClickIncrement' because its render mode 'Microsoft.AspNetCore.Components.Web.InteractiveWebAssemblyRenderMode' is not supported by interactive server-side rendering.</span>



## Utiliser la `CLI` : `-ai`

C'est l'option `--all-interactive` ou `-ai` qui par défaut est à `false`, qui passer à `true` ajoute le `@rendermode` dans `App.razor`.

```bash
dotnet new blazor -o BlazorServerAI -ai true
```

Ou pour une application par globalement en `webassembly`

```bash
dotnet new blazor -o BlazorAllInteractive -ai true -int webassembly
```

et

`App.razor`

```ruby
	// ...
	<HeadOutlet @rendermode="@InteractiveWebAssembly" />
</head>

<body>
    <Routes @rendermode="@InteractiveWebAssembly" />
```



