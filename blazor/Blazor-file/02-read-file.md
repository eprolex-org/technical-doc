# 02 Lecture d'un fichier



## `OpenReadStream`

`OpenReadStream` ne peu pas lire un fichier de plus de `500 Ko`.

```cs
IBrowserFile file = e.File;

var stream = file.OpenReadStream(maxAllowedSize: 600_000L);
```

Il ne vaut mieux pas essayer de copier le `stream` en mémoire avec `MemoryStream`.

- Charger le fichier du `client` directement vers un service du `serveur`.

