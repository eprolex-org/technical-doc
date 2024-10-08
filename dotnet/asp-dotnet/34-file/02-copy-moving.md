# 02 Copier et déplacer un `fichier`



## Copier un `fichier`

### récupérer le nom d'un `fichier` :  `Path.GetFileName`

```cs
string inputFileName = Path.GetFileName(InputFilePath);
```

Ceci va récupérer le `nom` et l'`extension` d'un `fichier` :

![input-file-name](assets/input-file-name.png)



### Créer le `path` de `backup` : `Path.Combine`

```cs
string backupFilePath = Path.Combine(backupDirectoryPath, inputFileName);
```



### Copier le `fichier` : `File.Copy(source, dest)`

```cs
WriteLine($"copying {inputFilePath} to {backupFilePath}");
File.Copy(InputPathFile, backupFilePath);
```

Si on relance le programme on a alors une `exception` :

<img src="assets/throwing-exception-file-already-exitsts.png" alt="throwing-exception-file-already-exitsts" style="zoom:50%;" />

#### Le `fichier` existe déjà !

On a un `overload` permettant d'écraser un fichier existant sans lancer une erreur :

<img src="assets/overload-copy-file.png" alt="overload-copy-file" style="zoom:50%;" />

### `File.Copy(source, dest, overwrite)`

> `overload` : méthodes de la même classe avec le même nom mais avec des paramètres et une valeur de retour différents (compile time static polymorphism).
>
> `override` Réécriture d'une méthode de la classe mère avec la même signature dans la classe fille. (Runtime dynamic polymorphism)

```cs
File.Copy(InputPathFile, backupFilePath, overwrite:true);
```

#### Attention ! Mon ancien `fichier` est écrasé.



## déplacer un `fichier` : 

## `File.Move(source, dest[, overwrite])`

```cs
// Ensure the directory exists
Directory.CreateDirectory(Path.Combine(rootDirectoryPath, InProgressDirectoryName));
string inProgressFilePath 
    = Path.Combine(rootDirectoryPath, InProgressDirectoryName, inputNameFile);

if(File.Exists(inProgressFilePath))
{
    WriteLine($"ERROR: a file with the name {inProgressFilePath} is already in the directory");
}

WriteLine($"Moving {InputFilePath} to {inProgressFilePath}");
File.Move(InputFilePath, inProgressFilePath);
```

Comme `Directory.CreateDirectory` ne lance pas d'exception, on peut le faire par défaut à chaque fois que le programme est exécuté.

`File.Move` permet aussi de copier un fichier sur un autre volume (disque réseau par exemple) :

```cs
var fileName = "test.png";

var startFolder = $"/Users/kms/Desktop/StartFolder/{fileName}";

// Disque réseau Windows (Samba)
var networkFolder = $"/Users/kms/w/Eprolex/networkFolder/{fileName}";

File.Move(startFolder, networkFolder);
```

`File.Move` utilise les méthodes du système d'exploitation pour soit renommer le fichier si on le déplace sur un même volume (très économe en ressource), soit copier et supprimer l'original si on copie le fichier sur un autre volume.

D'après `ChatGPT` `File.Move` est plus efficace dans ce. cas que l'utilisation de `FileStream`, notamment pour de gros fichiers.

> **Pour les petits fichiers**, la différence de performance entre `File.Move` et `FileStream` est négligeable.
>
> **Pour les gros fichiers**, surtout s'ils se trouvent sur le même volume, `File.Move` est beaucoup plus rapide et efficace, car il évite la lecture/écriture des données et se contente de modifier les métadonnées du système de fichiers.
>
> **Pour des volumes différents**, `File.Move` reste généralement plus efficace, car il délègue la gestion de la copie des données au système d'exploitation, qui peut être optimisé pour cette tâche.