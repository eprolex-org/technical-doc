# 21.5 Upload `File`

### Utilisation de `Results.File` et `IFormFile`

## `GET`:`Download` file from `API`

On va ici télécharger des fichier en récupérant le plus de méta-données.

<img src="assets/get-file-upload-schema-miro.png" alt="get-file-upload-schema-miro" />

## Utilisation de `GetAsync` et gestion de l'occupation Mémoire

On utilise `GetAsync` plutôt que `GetStreamAsync` pour pouvoir extraire du `content` le nom du fichier ainsi que le type `MIME`.

> ??? Je ne pense pas que ce soit possible avec `GetStreamAsync` qui renvoie directement un `stream` sans l'accès aux `headers` ???

De base on a le code suivant :

```cs
using var response = await client.GetAsync("/download");
```

Observation de la mémoire :

<img src="assets/memory-out-of-the-box-before-optimize.png" alt="memory-out-of-the-box-before-optimize" />

Le fichier est mis en cache côté `client`. Après trois fichiers on arrive dangereusement à `3Go` :

<img src="assets/danger-ram-occupation.png" alt="danger-ram-occupation" />

## Solution : `HttpCompletionOption`

On va dire à l'application que la requête est `done` à la fin de l'envoie des `Headers`, ainsi celle-ci ne sera pas mise en cache entièrement :

```cs
using var response = await client.GetAsync("/download", HttpCompletionOption.ResponseHeadersRead);
```

<img src="assets/memory-stable-version-client-http.png" alt="memory-stable-version-client-http" />

Seul les `headers` sont mis en `cache`, après trois téléchargements le poids mémoire n'a pas bougé. L'`API` (`PID: 50912`) reste aussi légère en mémoire.

> Afficher le `PID`
>
> ```cs
> Console.WriteLine($"Process Id: {Environment.ProcessId}");
> 
> app.Run();
> ```



### Code complet `Client` avec `GetAsync`

```cs
const string basePath = "/Users/hukar/RiderProjects/FileTransfertDemo";
var client = new HttpClient { BaseAddress = new Uri("http://localhost:5182") };

using var response = await client.GetAsync("/download", HttpCompletionOption.ResponseHeadersRead);
// using var response = await client.GetAsync("/download");

var filename = response.Content.Headers.ContentDisposition?.FileName ?? "unknow.txt";
var mimeType = response.Content.Headers.ContentType;

await using var fsWrite = new FileStream(
    $"{basePath}/files-uploaded/{TimeNow()}-{filename}",
    FileMode.Create
);

Console.WriteLine($"Start copy, type: {mimeType}");
Console.ReadKey();

await response.Content.CopyToAsync(fsWrite);

Console.WriteLine("Copy is done");

string TimeNow() => DateTime.Now.ToString("HH'h'mm'mn'ss's'");

```

On récupère le nom du fichier grâce à `response.Content.Headers.ContentDisposition.FileName`.

On récupère le `MIME Type` avec `response.Content.Headers.ContentType`.

Ce ne serait pas possible en utilisant `GetStreamAsync`.



### Code `API`

```cs
app.MapGet("/download", () =>
    File(
        $"{basePath}/files/{fileName}",
        mimeType,
        fileName
    ));
```

`Results.File` a un `overload` prenant le `Content Type` et le nom du fichier.



## `POST`:`Upload` file to `API`

<img src="assets/upload-file-to-api-schema-miro.png" alt="upload-file-to-api-schema-miro" />

Pour ce faire on doit simuler un formulaire `HTTP` `multipart/form-data`. Il existe dans ce but, une classe `MultipartFormDataContent` qui rempli correctement les `headers` de la requête et formate le `body`.

### `Http Client`

```cs
const string basePath = "/Users/hukar/RiderProjects/FileTransfertDemo";
const string fileName = "3601-bigfile.txt";
var client = new HttpClient { BaseAddress = new Uri("http://localhost:5182") };
```

```cs
var content = new MultipartFormDataContent();

await using var fsRead = new FileStream(
    $"{basePath}/files/{fileName}",
    FileMode.Open
);

var fileStream = new StreamContent(fsRead);

content.Add(fileStream, "file", fileName);

using var response = await client.PostAsync("/upload", content);
```

### `API`

```cs
app.MapPost("/upload", (IFormFile file) =>
{
    await using var fsWrite = new FileStream(
        $"/Users/hukar/RiderProjects/FileTransfertDemo/files-uploaded/{file.Name}",
        FileMode.Create
    );

    await file.CopyToAsync(fsWrite);

    return NoContent();
});
```

À ce stade on obtient une erreur :

<img src="assets/error-exception-broken-pipe-multipart-data-file-uploead.png" alt="error-exception-broken-pipe-multipart-data-file-uploead" />

Malgré que l'`exception` est lancée côté `client`, c'est le réglage de `multipartBodyLengthLimit` (par défaut `128 Mo`) côté `API` qui est en cause.

Il vaut mieux régler cette valeur `endpoint` par `endpoint` :

```cs
app.MapPost("/upload", (IFormFile file) =>
{
    // ...
})
.WithFormOptions(multipartBodyLengthLimit: 2_000_000_000L);
// default 128 MB
```

Cette fois l'erreur est côté `API` :

<img src="assets/antoforgery-problem-with-upload-formdata.png" alt="antoforgery-problem-with-upload-formdata" />

Le `endpoint` n'est pas protégé contre la falsification de formulaire, un `antiforgery token` devrait être passé.

Pour cet exemple je me contente de désactiver la protection contre la falsification :

```cs
app.MapPost("/upload", (IFormFile file) =>
{
    // ...
})
.WithFormOptions(multipartBodyLengthLimit: 2_000_000_000L)
.DisableAntiforgery();
```

Il faut aussi configurer `Kestrel` :

```cs
builder.WebHost.ConfigureKestrel(cfg =>
{
    // default 30 MB
    cfg.Limits.MaxRequestBodySize = 1_200_000_000L;
});
```

L'empreinte mémoire semble correcte.

> ## `Big File` et `IIS`
>
> https://stackoverflow.com/questions/38698350/increase-upload-file-size-in-asp-net-core
>
> If faut configurer `IIS` dans le fichier `web.config` :
>
> ```xml
> <system.webServer>
>     <security>
>         <requestFiltering>
>             <!-- Handle requests up to 1 GB -->
>             <requestLimits maxAllowedContentLength="1073741824" />
>         </requestFiltering>
>     </security>
> </system.webServer>
> ```
>
> Et dans son application `asp.net` :
>
> ```cs
> services.Configure<IISServerOptions>(options =>
> {
>     options.MaxRequestBodySize = int.MaxValue;
> });
> ```
>
> 
