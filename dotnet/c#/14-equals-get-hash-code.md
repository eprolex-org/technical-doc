# 14 `Equals` et  `GetHashCode`

# `Equals`

## Point de départ

```cs
public class EnumerationClasse
{
    public string Propriete1 { get; set; }
    public string Propriete2 { get; set; }
    public string Propriete3 { get; set; }

    public override bool Equals(object obj)
    {
        if (obj is EnumerationClasse autre)
        {
            return Propriete1 == autre.Propriete1 &&
                   Propriete2 == autre.Propriete2 &&
                   Propriete3 == autre.Propriete3;
        }
        return false;
    }
```

Notre égalité est basée sur la valeur des trois propriété, on doit faire correspondre `GetHashCode`

> ## Version minimaliste de `Equals`
>
> ```cs
> public override bool Equals(object obj)
>     => obj is EnumerationClasse autre &&
>         Propriete1 == autre.Propriete1 &&
>         Propriete2 == autre.Propriete2 &&
>         Propriete3 == autre.Propriete3;
> ```
>
> 



## `IEquatable<T>` et `Equals(T obj)`

Lors d'un test avec un objet de même type, `Equals(T)` offre de meilleur performance que `Equals(object)`. 

La raison en est qu'il n'y a pas besoin de` test` et de `cast` sur le deuxième élément.

```cs
public bool Equals(EnumerationClasse? other)
{
    if (ReferenceEquals(null, other)) return false; // 1
    if (ReferenceEquals(this, other)) return true; // 2
    
    return  Propriete1 == autre.Propriete1 && // 3
            Propriete2 == autre.Propriete2 &&
            Propriete3 == autre.Propriete3;
}
```

1. Si le deuxième élément est `null` on retourne `false`
2. si le deuxième élément est moi-même on retourne `true`
3. On regarde l'égalité de chaque propriété entrant dans l'égalité

On doit maintenant `override` `Equals(object)`:

```cs
public override bool Equals(object? obj)
        =>  ReferenceEquals(this, obj) || 
    		obj is EnumerationClasse other && Equals(other);
```



## `==` et `!=`

On peut vouloir surcharger ces opérateurs pour les rendre cohérents avec `Equals`.

```cs
public static bool operator ==(EnumerationLibelle left, EnumerationLibelle right)
    => left is not null && left.Equals(right);

public static bool operator!=(EnumerationLibelle left, EnumerationLibelle right)
    =>!(left == right);
```

Si `left` est `null`, la comparaison `left is not null` renvoie `false`.

Si `right` est `null`, `left.Equals(right)` retourne `false` à sa première ligne :

```cs
public bool Equals(EnumerationClasse? other)
{
    if (ReferenceEquals(null, other)) return false;
```





# `GetHashCode`

Lorsqu'on `override` la méthode `Equals`, on doit faire correspondre la méthode `GetHashCode`pour que celle-ci puisse retrouver l'objet sur les mêmes critères que l'égalité.

## Le plus simple : `Combine`

```cs
public override int GetHashCode()
{
    return HashCode.Combine(Propriete1, Propriete2, Propriete3);
}
```

La méthode s'occupe de gérer les valeurs `null` et garantie une distribution correcte des valeurs de `Hashage`.



## Avec un `Tuple`

```cs
public override int GetHashCode()
{
    return (Propriete1, Propriete2, Propriete3).GetHashCode();
}
```

Qui utilise `HashCode.Combine` sous le capot :

```cs
/// <summary>
/// Returns the hash code for the current <see cref="ValueTuple{T1, T2, T3, T4}"/> instance.
/// </summary>
/// <returns>A 32-bit signed integer hash code.</returns>
public override int GetHashCode()
{
    return HashCode.Combine(Item1?.GetHashCode() ?? 0,
                            Item2?.GetHashCode() ?? 0,
                            Item3?.GetHashCode() ?? 0,
                            Item4?.GetHashCode() ?? 0);
}
```



## En combinant les différents `HashCode` avec l'opérateur `^`

```cs
public override int GetHashCode()
{
    return (Propriete1.GetHashCode() * 397) 
        ^ Propriete2.GetHashCode() 
        ^ Propriete3.GetHashCode();
}
```



## À la main

En utilisant des nombres premiers pour éviter les collisions de valeur :

```cs
public override int GetHashCode()
{
    unchecked
    {
        int hash = 17; // Choisir un nombre premier comme point de départ.
        hash = hash * 23 + (Propriete1?.GetHashCode() ?? 0);
        hash = hash * 23 + (Propriete2?.GetHashCode() ?? 0);
        hash = hash * 23 + (Propriete3?.GetHashCode() ?? 0);
        return hash;
    }
}
```

> ### `unchecked` (source: ChatGPT)
>
> Le bloc `unchecked` en `C#` est utilisé pour indiquer au compilateur de désactiver la vérification des dépassements d'entiers pour les opérations effectuées à l'intérieur de ce bloc. Cela signifie que si une opération entraîne un dépassement d'entier (par exemple, un débordement de capacité d'un type numérique), aucune exception `OverflowException` ne sera levée, et le calcul continuera avec des valeurs inattendues en cas de dépassement.
>
> Dans le contexte de la méthode `GetHashCode` que j'ai présentée précédemment, l'utilisation de `unchecked` est principalement une mesure de sécurité. Elle indique explicitement au compilateur que les dépassements d'entiers dans les opérations de hachage (par exemple, la multiplication par 397) sont attendus et ne doivent pas générer d'exceptions en cas de dépassement. Cela permet d'obtenir une meilleure performance, car les exceptions ne sont pas levées pour chaque opération de hachage.

