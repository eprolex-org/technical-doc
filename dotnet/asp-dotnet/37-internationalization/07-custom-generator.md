# 07 un `générateur` custom

Pour éviter d'utiliser les fichiers `resx`, il peut être intéressant de créer son propre générateur de classes de traduction.

## Point de départ

De manière général, il est simple de gérer son système de traduction en partant d'un fichier `Excel` (tout le monde a `Excel`) pouvant être fournis au service chargé de la traduction :

<img src="assets/excel-working-trraduction-handle-iiuddfscwwnjhd.png" alt="excel-working-trraduction-handle-iiuddfscwwnjhd" />

Dans la version `Desktop Mac`,  dans `File/Save As`  :

<img src="assets/sab-ve-as-excel-tab-txt-yyuiejrretz.png" alt="sab-ve-as-excel-tab-txt-yyuiejrretz" />

je peux choisir de sauvegarder au format `tab-delimited text` .

> ### Pourquoi utiliser `tab` plutôt que la virgule ou le point-virgule ?
>
> Il pourrait se trouver une virgule ou un point-virgule dans le texte même de la traduction, cela générerai une erreur.
>
> Avec `tab` , le texte peut contenir des virgules ou des points-virgules (bien sûr `tab` doit être interdit dans les textes).



## Programme de génération d'une classe `DemandeAvisTranslation`

<img src="assets/tree-of-diretctories-translation-sshhgfdfqklbx.png" alt="tree-of-diretctories-translation-sshhgfdfqklbx" />

Un fichier `DemandeAvis.txt` (venant de `Excel`) est déposé dans le dossier `Translations`.

```cs
var fileName = "DemandeAvis";

var basePath = Path.Combine(Directory.GetCurrentDirectory(), "..", "..", "..");

var filePath = Path.Combine(basePath, "Translations", $"{fileName}.txt");
var fileUTF8Path = Path.Combine(basePath, "Translations", $"{fileName}UTF8.txt");

// 0. Convertir le charset en UTF8
// Gère les charsets non-unicode
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
ConvertCharsetToUtf8(filePath, fileUTF8Path);
Console.WriteLine("fichier converti en UTF8 ✅");
```

```cs
void ConvertCharsetToUtf8(string inPath, string outPath)
{
    string contentFinal = File.ReadAllText(inPath, Encoding.GetEncoding("macintosh"));
    
    File.WriteAllText(outPath, contentFinal, new UTF8Encoding(false));
}
```

Le fichier est converti en `UTF8`.

```cs
// 1. ouvrir le fichier  translation.txt
var lines = await File.ReadAllLinesAsync(fileUTF8Path);

// 2. Si le tableau a moins de 2 lignes alors on quitte
if (lines.Length <= 1) return;
```

La première ligne contient les en-têtes (noms de colonne).

```cs
// 3. écrire le début du fichier à générer
var sb = new StringBuilder();

sb.Append(
    $$"""
      using System.Globalization;

      namespace TranslationGenerator.GeneratedClasses;

      public static class {{fileName}}Translation
      {
      
      """
);

// 4. Pour chaque ligne (sauf la première qui contient le nom des colonnes)
// généré soit une propriété, soit une méthode si argument(s)
for (var i = 1; i < lines.Length; i++)
{
    var lineElements = lines[i].Split("\t");
    
    // PropertyName [0] | Arguments [1]  | Fr [2] | Nl [3] | De [4]
    var propertyName = lineElements[0];

    var arguments = lineElements[1];

    var textFr = string.IsNullOrEmpty(lineElements[2]) ? "No translation for the moment ..." : lineElements[2];
    var textNl = string.IsNullOrEmpty(lineElements[3]) ? textFr : lineElements[3];
    var textDe = string.IsNullOrEmpty(lineElements[4]) ? textFr : lineElements[4];


    if (!string.IsNullOrEmpty(arguments))
    {

        sb.Append(
            $$"""
                  public static string {{propertyName}}({{arguments}}) => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
                  {
                      "fr" => $"{{textFr}}",
                      "nl" => $"{{textNl}}",
                      "de" => $"{{textDe}}",
                      _ => $"{{textFr}}"
                  };
              """
        );
    }
    else
    {
        sb.Append(
            $$"""
                  public static string {{propertyName}} => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
                  {
                      "fr" => "{{textFr}}",
                      "nl" => "{{textNl}}",
                      "de" => "{{textDe}}",
                      _ => "{{textFr}}"
                  };
              """
        );
    }

    sb.Append("\n\n");
}

// 5. Fermer la classe et obtenir le string complet
sb.Append("""
            }
            """);

var contentString = sb.ToString();
var fileOutputPath = Path.Combine(basePath, "GeneratedClasses", $"{fileName}Translation.cs");
```

Code générant la classe de traduction à l'aide d'un `StringBuilder`.

Les doubles `$$` permettent d'écrire des `{` de manière littérale dans le texte. L'interpolation utilise donc deux accolades  `{{ ... }}`.

```cs
// 6. Générer le fichier
await File.WriteAllTextAsync(fileOutputPath, contentString);

Console.WriteLine($"{fileName} is created ✅");
```

Le fichier `DemandeAvisTranslations.cs` est généré.



## `DemandeAvisTranslation.cs` (exemple)

```cs
using System.Globalization;

namespace TranslationGenerator.GeneratedClasses;

public static class DemandeAvisTranslation
{
    public static string CreatedDate(DateTime creationLe, string utilisateur) => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
    {
        "fr" => $"Création le {creationLe} Création par {utilisateur}",
        "nl" => $"Création le {creationLe} Création par {utilisateur}",
        "de" => $"Création le {creationLe} Création par {utilisateur}",
        _ => $"Création le {creationLe} Création par {utilisateur}"
    };

    public static string DepotDate(DateTime depotLe, string utilisateur) => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
    {
        "fr" => $"Dépôt le {depotLe} Dépôt par {utilisateur}",
        "nl" => $"Dépôt le {depotLe} Dépôt par {utilisateur}",
        "de" => $"Dépôt le {depotLe} Dépôt par {utilisateur}",
        _ => $"Dépôt le {depotLe} Dépôt par {utilisateur}"
    };

    public static string ModifiedDate(DateTime modificationLe, string utilisateur) => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
    {
        "fr" => $"Modification le {modificationLe} Modification par {utilisateur}",
        "nl" => $"Modify on {modificationLe} Modify By par {utilisateur}",
        "de" => $"Pas de traduction en allemand",
        _ => $"Modification le {modificationLe} Modification par {utilisateur}"
    };

    public static string EstConjointe => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
    {
        "fr" => "Est conjointe",
        "nl" => "Est conjointe",
        "de" => "Est conjointe",
        _ => "Est conjointe"
    };

    public static string MandatoryField => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
    {
        "fr" => "Champ obligatoire",
        "nl" => "Champ obligatoire",
        "de" => "Champ obligatoire",
        _ => "Champ obligatoire"
    };

    public static string LoaderDemandeAvisDeleting => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
    {
        "fr" => "La demande d'avis est en train d'être supprimée ...",
        "nl" => "La demande d'avis est en train d'être supprimée ...",
        "de" => "La demande d'avis est en train d'être supprimée ...",
        _ => "La demande d'avis est en train d'être supprimée ..."
    };

    public static string DialogDemandeAvisDelete(int demandeAvisId) => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
    {
        "fr" => $"Voulez-vous supprimer <br> la Demande d'avis d'Id [{demandeAvisId}] ?",
        "nl" => $"Voulez-vous supprimer <br> la Demande d'avis d'Id [{demandeAvisId}] ?",
        "de" => $"Voulez-vous supprimer <br> la Demande d'avis d'Id [{demandeAvisId}] ?",
        _ => $"Voulez-vous supprimer <br> la Demande d'avis d'Id [{demandeAvisId}] ?"
    };

    public static string SnackBarDemandeAvisNotDeleted(int demandeAvisId) => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
    {
        "fr" => $"La demande Avis Id [{demandeAvisId}] n'a pas pu être supprimée !!!",
        "nl" => $"La demande Avis Id [{demandeAvisId}] n'a pas pu être supprimée !!!",
        "de" => $"La demande Avis Id [{demandeAvisId}] n'a pas pu être supprimée !!!",
        _ => $"La demande Avis Id [{demandeAvisId}] n'a pas pu être supprimée !!!"
    };

    public static string DemandeAvisDeleteBtn => CultureInfo.CurrentUICulture.TwoLetterISOLanguageName switch
    {
        "fr" => "Supprimer la demande d'avis",
        "nl" => "Delete Demande Avis",
        "de" => "Delete Demande Avis (allemand)",
        _ => "Supprimer la demande d'avis"
    };

}
```



