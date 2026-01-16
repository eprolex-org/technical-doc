# 15. L'attribut `[AsParameters]`

Permet de mapper des `queryParams` dans un objet (Typage fort).

Url:

```http
/deposant?Tri=AlphabeticalAsc&DemandeurIds=3&DemandeurIds=2
```



```cs
public class DeposantCriteresDto
{
    public string? Recherche { get; set; }
    public DeposantTri Tri { get; set; } = DeposantTri.AlphabeticalAsc;
    public int[] DemandeurIds { get; set; } = [];
}
```

```cs
app.MapGet("/deposant", async (
                [AsParameters] DeposantCriteresDto criteresDto
            ) => { /* ... */ }
          );
```

`[AsParameter]` ne binde (mappe) pas avec `IEnumerable<int>` mais seulement avec un type "simple" comme `int[]`.