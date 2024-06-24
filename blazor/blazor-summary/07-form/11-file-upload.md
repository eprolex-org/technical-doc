# 11 `File` Upload

## `InputFile`

Le composant `InputFile` nous permet de télécharger un fichier dans Blazor :

```html
<InputFile OnChange="OnChangeHandle"/> <br/>
```

L'événement `onchange` reçoit un `InputFileChangeEventArgs` qui entre autre contient soit un `IBrowserFile` soit une liste `IReadOnlyList<IBrowserFile>`.

### `IBrowserFile`

```cs
public interface IBrowserFile
{
    string Name { get; }
    DateTimeOffset LastModified { get; }
    long Size { get; }
    string ContentType { get; }

    Stream OpenReadStream(long maxAllowedSize = 512000, CancellationToken cancellationToken = default (CancellationToken));
}
```

C'est la méthode `OpenReadStream` qui va nous permettre de récupérer (`uploader`) le fichier.