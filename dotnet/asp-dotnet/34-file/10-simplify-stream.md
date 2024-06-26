# 10 simplifier l'écriture de `Stream`

On peut simplifier l'écriture des `stream` en utilisant la classe `File` :

```cs
using StreamReader reader = File.OpenText(startPath);
```

au lieu de 

```cs
var optionsRead = new FileStreamOptions
{
    Mode = FileMode.Open,
    Access = FileAccess.Read
};
using var fsRead = new FileStream(startPath, optionsRead);
using var reader = new StreamReader(fsRead);
```

et

```cs
using var writer = new StreamWriter(destinationPath);
```

au lieu de 

```cs
var optionsWrite = new FileStreamOptions
{
    Mode = FileMode.Create,
    Access = FileAccess.Write
};
using var fsWrite = new FileStream(destinationPath, optionsWrite);
```



## Programme simplifier

```cs
using StreamReader reader = File.OpenText(startPath);

using var writer = new StreamWriter(destinationPath);

while (!reader.EndOfStream)
{
    writer.WriteLine(reader.ReadLine());
}
```

> Il y a quelques subtilités si on veut éviter d'avoir une ligne supplémentaire en fin de fichier :
>
> ```cs
> while (!reader.EndOfStream)
> {
>     var line = reader.ReadLine()
>     
>     bool isLastLine = reader
>     if(isLastLine)
>     {
>         writer.Write(line);
>     }
>     else
>     {
>         writer.WriteLine(line);
>     } 
> }
> ```