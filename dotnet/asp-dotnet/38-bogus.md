# 38 `Bogus`

```bash
dotnet add package Bogus
```



## Utilisation simple

On crée sa classe de génération :

`Bot.cs`

```cs
public class Bot
{
    public int SerialNumber { get; set; }
    public string Name { get; set; }
    public int Power { get; set; }
    public int Speed { get; set; }
}
```

Puis dans `Program.cs`

```cs
using Bogus;
// ...

var botFake = new Faker<Bot>()
    .StrictMode(true)
    .RuleFor(b => b.SerialNumber, f => f.IndexFaker)
    .RuleFor(b => b.Name, f => f.Vehicle.Model())
    .RuleFor(b => b.Power, f => f.Random.Number(10, 100))
    .RuleFor(b => b.Speed, f => f.Random.Number(1, 20));

var bots = botFake.Generate(4).ToList();
```

`StricMode` s'assure que toutes les propriétés ont un `RuleFor`.

Méthode d'extension `Dump`:

```cs

static class ExtensionsHelpers
{
    public static void Dump(this object obj)
        => Console.WriteLine(JsonSerializer.Serialize(
            obj,
            new JsonSerializerOptions { WriteIndented = true }));
}
```

Affichage du résultat :

```cs
bots.Dump();
```

```json
[
  {
    "SerialNumber": 0,
    "Name": "Aventador",
    "Power": 10,
    "Speed": 4
  },
  {
    "SerialNumber": 1,
    "Name": "Altima",
    "Power": 10,
    "Speed": 8
  },
  {
    "SerialNumber": 2,
    "Name": "Corvette",
    "Power": 67,
    "Speed": 5
  },
  {
    "SerialNumber": 3,
    "Name": "XC90",
    "Power": 31,
    "Speed": 20
  }
]
```

On observe que `Random.Number` fonctionne avec inclusion des valeurs : `[minValue, maxValue]`.