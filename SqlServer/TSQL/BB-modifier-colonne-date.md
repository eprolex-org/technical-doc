# Modifier une colonne de `datetime2`

## Problématique

Ayant passé les différentes colonnes de date en `datetime(0)` au lieu de `datetime2(7)`, j'ai eu des soucis dans ce que me renvoyaient des `SELECT`.


Imaginons un `UPDATE` qui met une date dans une colonne `FinValidite` par exemple `2025-12-09 15:21:13`.
Juste après un `SELECT` a le `WHERE` suivant :

```sql
WHERE dep.FinValidite IS NULL OR dep.FinValidite > @now
```

et que `@now` vaille lui aussi `2025-12-09 15:21:13` quelques millisecondes plus tard. Le `SELECT` renverra un enregistrement qui n'est plus valide.

Si on augment la précision de `datetime2`, on résout ce problème d'indétermination du retour de `SELECT` car on est à la centaine de nanosecondes près.

On peut aussi mettre une `FinValidite` légèrement dans le passé :
```sql
DECLARE @finValidite datetime2 = DATEADD(SECOND, -1, SYSDATETIME())
```



## Correction des colonnes `datetime2`

J'ai dû remettre toutes les dates à `datetime2(7)`, ce qui est la valeur par défaut de `datetime2` est qui équivaut à `100 ns` (10exp-7 seconde = 0,0000001 seconde).

### Recherche de toutes les colonnes de type `datetime2(0)`

```sql
SELECT
    s.name  AS [Schema],
    t.name  AS [Table],
    c.name  AS [Column],
    c.is_nullable,
    ty.name AS [Type],
    c.scale
FROM sys.columns c
         JOIN sys.tables t  ON t.object_id = c.object_id
         JOIN sys.schemas s ON s.schema_id = t.schema_id
         JOIN sys.types ty  ON ty.user_type_id = c.user_type_id
WHERE ty.name = 'datetime2'
  AND c.scale = 0
ORDER BY s.name, t.name, c.column_id;
```

Si ces colonnes ne font pas parties d'Index ou autres contraintes, on peut généré le script d'update comme ceci :
```sql
DECLARE @sql nvarchar(max) = N'';

SELECT @sql +=
       N'ALTER TABLE ' + QUOTENAME(s.name) + N'.' + QUOTENAME(t.name) +
       N' ALTER COLUMN ' + QUOTENAME(c.name) + N' datetime2(7) ' +
       CASE WHEN c.is_nullable = 1 THEN N'NULL' ELSE N'NOT NULL' END +
       N';' + CHAR(13) + CHAR(10)
FROM sys.columns c
         JOIN sys.tables t  ON t.object_id = c.object_id
         JOIN sys.schemas s ON s.schema_id = t.schema_id
         JOIN sys.types ty  ON ty.user_type_id = c.user_type_id
WHERE ty.name = 'datetime2'
  AND c.scale = 0;

PRINT @sql;

-- EXEC sp_executesql @sql;
```

Si une contrainte bloque, il faut d'abord la supprimer, modifier la colonne et remettre la contrainte.