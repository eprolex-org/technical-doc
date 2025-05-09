# 08 Détecter le `charset`



## Détection du `charset` d'un fichier

### `CLI`

Le fichier obtenu n'est pas en `UTF-8` et les accents sont remplacés par des losanges.

Avec l'utilitaire `uchardet` (installé avec `brew install uchardet`) je peux connaitre l'encodage d'un fichier :

```bash
uchardet Desktop/Traducton\ Eprolex.txt
ISO-8859-3
```

> `u char det` est l'acronyme de `Universal Charset Detector`.



### `.net`

Il existe aussi deux librairies en `.net`  pour déterminer le `charset`:

#### `Ude`

```cs
using var fs = File.OpenRead(filePath);

Ude.CharsetDetector charsetDetector = new();

charsetDetector.Feed(fs);
charsetDetector.DataEnd();

if (charsetDetector.Charset != null)
{
    Console.WriteLine("Charset: {0}, confidence: {1}",
        charsetDetector.Charset, charsetDetector.Confidence);
}
else
{
    Console.WriteLine("Detection failed.");
}
```

La liste de `charset` pouvant être détecté est assez réduite.



#### `UTF.Unknown`

La liste est bien plus conséquente.

```cs
DetectionResult result = CharsetDetector.DetectFromFile(filePath);

// Get the best Detection
DetectionDetail resultDetected = results.Detected;

// Get the alias of the found encoding
string encodingName = resultDetected.EncodingName;

// Get the System.Text.Encoding of the found encoding (can be null if not available)
Encoding encoding = resultDetected.Encoding;

// Get the confidence of the found encoding (between 0 and 1)
float confidence = resultDetected.Confidence;

Console.WriteLine($"encoding: {encodingName}, confidence: {confidence}");
```

```
encoding: iso-8859-3, confidence: 0,4755236
```



On peut aussi voire toutes les propositions (s'il y en a plus d'une)

```cs
DetectionResult result = CharsetDetector.DetectFromFile(filePath);
IList<DetectionDetail> allDetails = result.Details;

foreach (var detail in allDetails)
{
    encodingName = detail.EncodingName;
    
    confidence = detail.Confidence;

    Console.WriteLine($"encoding: {encodingName}, confidence: {confidence}");
}
```



> ## Problème du fichier généré par `Excel`
>
> Il semble que `ISO-8859-3` ne soit pas le `charset` réel du fichier car le problème des caractères accentués persiste.
>
> Pour pouvoir tester tout type de `charset`, il faut ajouter la ligne :
>
> ```cs
> Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
> ```
>
> Cela permet d'activer les encodages `non Unicode`.
>
> On peut maintenant tester "manuellement" différent `encoding` en regardant dans la `console` le rendu des accents :
>
> ```cs
> string[] encodingsToTry = ["Windows-1252", "ISO-8859-1", "ISO-8859-3", "macintosh" ];
> 
> foreach (var enc in encodingsToTry)
> {
>  var encoding = Encoding.GetEncoding(enc);
>  Console.WriteLine(encoding.EncodingName);
> 
>  string content = File.ReadAllText(fileInPath, encoding);
>  Console.WriteLine(content[..100]);
> }
> ```
>
> ```
> Western European (Windows), encoding: Windows-1252
> CreatedDate     DateTime creationLe, string utilisateur CrŽation le {creationLe} CrŽation par {utilisate
> 
> Western European (ISO), encoding: ISO-8859-1
> CreatedDate     DateTime creationLe, string utilisateur Cration le {creationLe} Cration par {utilisate
> 
> Latin 3 (ISO), encoding: ISO-8859-3
> CreatedDate     DateTime creationLe, string utilisateur Cration le {creationLe} Cration par {utilisate
> 
> Western European (Mac), encoding: macintosh
> CreatedDate     DateTime creationLe, string utilisateur Création le {creationLe} Création par {utilisate
> ```



## Transformation du `charset` 

```cs
var fileText = File.ReadAllText(filePath, originalEncoding);

File.WriteAllText(fileOutPath, fileText, Encoding.UTF8);
```

On doit préciser le `originalEncoding` pour que `fileText` ne soit pas altéré.

Dans notre cas on a :

```cs
string contentFinal = File.ReadAllText(fileInPath, Encoding.GetEncoding("macintosh"));
File.WriteAllText(fileOutPath, contentFinal, new UTF8Encoding(false));
```

`new UTF8Encoding(false)` ne rajoute pas de `BOM`  au début du fichier contrairement à `Encoding.UTF8` qui ajoute un `BOM`.
