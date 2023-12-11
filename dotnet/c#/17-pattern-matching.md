# 17 Évolution du `Pattern Matching`

## `c# 1` à `c# 6`

Pour tester un type et la valeur d'une propriété on écrivait :

```cs
if (orange.GetType() == typeof(Orange) && orange.Color == Color.Orange)
{
    Console.WriteLine("This is an orange orange");
}
```



## `c# 7`

On peut utiliser un `switch`.

```cs
foreach (var fruit in fruits)
{
    switch (fruit)
    {
        case Banana banana when banana.Color == Color.Green:
            Console.WriteLine("A green banana");
            break;
        case Orange orange when orange.Color == Color.Red:
            Console.WriteLine("A sanguine orange");
            break;
        default:
            Console.WriteLine("Some other fruit");
            break;
    }
}
```



## `c# 8` Switch Expression

On peut maintenant construire un résultat avec un `switch`.

```cs
foreach (var fruit in fruits)
{
    var result = fruit switch
    {
        Orange orange when orange.Color == Color.Orange 
            => "I'm an orange orange",
            
        Banana banana when banana.Color == Color.Orange 
            => "I'm a banana orange [Toxic]",
            
        Orange orange when orange.Color == Color.Green 
            => "I'm an orange green",
            
        _ => "I'm some other fruit"
    };

    Console.WriteLine(result);
}
```

On utilise toujours le `when` pour ajouter une condition.



## Capturer le résultat du `matching`

On peut déclarer une variable pour recevoir le résultat du matching :

### Je ne capture pas le résultat

```cs
var result = fruit switch
{
    Orange { WeightInGrams: <70 and >50, Color: Color.Orange } => "I'm a calibrate orange orange",
    Orange => "I'm an orange",
```

### Je capture le résultat

```cs
var result = fruit switch
    {
        Orange { WeightInGrams: <70 and >50, Color: Color.Orange } result => $"I'm a calibrate orange orange {result.Brand}",
        Orange result => $"I'm an orange {result.Brand}",
```

Cela dépend si on a besoin des valeurs de l'élément matché pour le retour.



## Les `Patterns` en `c#`

### `Type` pattern

Va vérifier le `type` et aussi qu'il n'est pas `null`.

```cs
var orange1 = new Orange(Color.Green);
var orange2 = new Orange(Color.Orange);

Banana? banana1 = null; // <- ici null
var banana2 = new Banana(Color.Orange);

Fruit[] fruits = [ banana1, orange1, banana2, orange2];


foreach (var fruit in fruits)
{
    var result = fruit switch
    {
        Orange => "I'm an orange",
        Banana => "I'm a banana",
        _ => "I'm some other fruit or null"
    };

    Console.WriteLine(result);
}
```

```
I'm some other fruit or null
I'm an orange
I'm a banana
I'm an orange
```

On utilise le `discard` : `_` en remplacement de `default`.

`_` peut matcher avec n'importe quoi (`null` compris).

On pourrait capturer la valeur inconnue comme ceci :

```cs
var result = fruit switch
{
    // ...
    var something => "I'm some other fruit or null"
};
```

Ceci peut-être dangereux car `var something ` match aussi avec `null`.

Cet exemple provoque une exception :

```cs
var orange1 = new Orange(Color.Green);
var orange2 = new Orange(Color.Orange);

Banana? banana1 = null;
var banana2 = new Banana(Color.Orange);

Fruit[] fruits = [ banana1, orange1, banana2, orange2];

foreach (var fruit in fruits)
{
    var result = fruit switch
    {
        Orange => "I'm an orange",
        Banana  => "I'm a banana",
        var something => $"I'm some other fruit or null {something.Color}"
    };

    Console.WriteLine(result);
}
```

```
Unhandled exception. System.NullReferenceException: Object reference not set to an instance of an object.
```



### `Positional` pattern

Utilisé avec la `Deconstruct Method`.



### `Tuple` pattern



### `Property` pattern et `Relational` pattern

Plus grand ou plus petit que.

```cs
var result = fruit switch
    {
        Orange { WeightInGrams: <70 and >50 } => "I'm a calibrate orange",
        Orange => "I'm an orange",
        _ => $"I'm some other fruit or null"
    };
```

Avec deux `Property` :

```cs
var result = fruit switch
{
    Orange { WeightInGrams: <70 and >50, Color: Color.Orange } 
        => "I'm a calibrate orange orange",
    Orange => "I'm an orange",
    _ => $"I'm some other fruit or null"
};
```

On peut aussi utiliser la `Property` seule :

```cs
var result = fruit switch
{
    { WeightInGrams: <70 and >50, Color: Color.Orange } f 
        => $"I'm a calibrate fruit orange {f.Brand}",
```

Ici on ne déclare pas le type.

### `if (result is { })`

`{ }` match avec une `instance`.

Dans un `switch` on écrira :

```cs
var result = fruit switch
{
        { } => "It's an instance",
```





### Conjonctif `"and"` pattern



### Disjonctif `"or"` pattern



### `List` pattern

```cs
string[] payloads = [ "orange", "banana", "cherry", "ananas" ];
```

```cs
// list pattern
if(payloads is ["orange", ..]) 
    Console.WriteLine("first element is orange");
```

```
first element is orange
```

---

```cs
// slice pattern
if(payloads is ["orange", _, .. var slice]) 
    Console.WriteLine($"slice element one {slice[0]}");
```

```
slice element one cherry
```





### `Parenthesized` pattern



### Négation `"not"` pattern



### `Recursive` pattern

 

