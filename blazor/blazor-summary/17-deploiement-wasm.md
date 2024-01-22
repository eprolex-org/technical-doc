# 17 Publier et déployer un site `Blazor Wasm` sur `IIS`



## Installer les outils de `build`

Il faut les droits admin

```bash
sudo dotnet workload install wasm-tools
```



## Publier l'application `publish`

```bash
dotnet publish -c Release
```

<img src="assets/Screenshot%202023-10-19%20at%2011.07.32-7706527.png" alt="Screenshot 2023-10-19 at 11.07.32" style="zoom:80%;" />

On trouve l'application dans le dossier `publish`.



## `IIS`

Il suffit de créer un site et de déposer `wwwroot` et `web.config` dans un dossier de `IIS`.

> On a besoin d'installer sur `IIS` l'extension `url-rewrite`
>
> https://www.iis.net/downloads/microsoft/url-rewrite