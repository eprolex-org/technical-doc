# 02 Les `record`

## `Record` et `DTO`

Ce sont de bon candidat pour les `DTO`

```cs
public record DtoBase (
    int Id, 
    DateTimeOffset Date, 
    string Name, 
    EventType EventType, 
    string Venue
);
```

Plus rapide à écrire qu'une classe.

## Héritage

```cs
public record Fruit(Color Color);

public record Orange(Color Color) : Fruit(Color);

public record Banana() : Fruit(Color.Yellow);
```

