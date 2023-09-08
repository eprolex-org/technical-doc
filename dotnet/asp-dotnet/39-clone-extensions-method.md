# 39 Clone un objet

Pour `cloner` un objet, une technique est de sérializer et déserializer un objet.

On peut créer une `méthode d'extension` au niveau de son application pour ajouter cette fonctionnalité.



## Utilisation de `system.Text.Json`

> Proposition de `ChatGPT`

```cs
public static class UtilitiesExtensions
{
    public static T DeepClone<T>(this T source) where T : class
    {
        
        // Utilise System.Text.Json pour sérialiser l'objet source en JSON
        string json = JsonSerializer.Serialize(source);

        // Désérialise le JSON pour créer une copie profonde
        T clone = JsonSerializer.Deserialize<T>(json) ?? (T)new object();

        return clone;
    }
}
```

Le code a été amélioré en supprimant la vérification ci-dessous et en ajoutant une contrainte `where T: class` qui dans un contexte de `nullable` n'accepte que des types non `nullable`.

```cs
if (source == null)
{
    throw new ArgumentNullException(
        nameof(source), 
        "Cannot clone a null object."
    );
}
```



## gestion d'une valeur `null`

On peut peut-être lancer une `exception` plutôt qu'un objet vide :

```cs
public static T DeepClone<T>(this T source) where T : class
{
    var serialized = JsonSerializer.Serialize(source);
    return 
        JsonSerializer.Deserialize<T>(serialized) 
        ?? throw new NullReferenceException();
}
```

> Version en une ligne un peu soufflée par `Tabnine`
>
> ```cs
> static class CloningExtensions
> {
>     public static T Clone<T>(this T obj) where T : class
>         =>  JsonSerializer.Deserialize<T>(JsonSerializer.Serialize(obj))
>             ?? throw new Exception("Failed to clone object");  
> }
> ```
