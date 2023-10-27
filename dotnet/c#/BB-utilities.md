# BB Trousse à outils

## Générer une liste d'entiers consécutifs : `Enumerable.Range`

```cs
foreach (var nb in Enumerable.Range(40, 80))
{
    Console.Write(nb + " : ");
    Console.WriteLine((char)nb);
}
```

