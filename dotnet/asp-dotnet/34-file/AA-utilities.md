# AA Utilitaires

## `Path` et `Directory`

#### ! Toujours utiliser `Path.Combine()` pour créer un chemin.

### `Directory.GetCurrentDirectory()`

```cs
var path = Path.Combine(Directory.GetCurrentDirectory());

// /Users/hukar/RiderProjects/FileApiTestAndExperiment/FileApiTestAndExperiment/bin/Debug/net8.0
```

On obtient la même chose avec `Environment.CurrentDirectory`.



## `File` et `FileStream`

```cs
FileStream fs = File.Create(filePath);
```

`FileStream` travaille avec des tableaux d'octets.

Si on veut travailler avec du `texte`, on utilise alors les classes `StreamReader` et `StreamWriter`.



### Écrire du texte dans un fichier `streamWriter.WriteLine`

```cs
FileStream fs = File.Create(filePath);

var sw = new StreamWriter(fs);

for (int i = 1; i <= 10; i++)
{
    sw.WriteLine($"My line number {i}");
}

sw.Close();
```

#### ! `sw.Close` est obligatoire sinon le fichier est vide.

Je peux aussi simplement utiliser `using` :

```cs
using var sw = new StreamWriter(fs);
```



### Lire dans un fichier texte `StreamReader`

```cs
using var sr = new StreamReader(filePath);

while (!sr.EndOfStream)
{
    var output = sr.ReadLine();
    // do something
}
```



### Lire en une fois

```cs
using var sr = new StreamReader(filePath);

var output = sr.ReadToEnd();
```

