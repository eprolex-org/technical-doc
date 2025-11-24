# 02 Les `Types` et les `Champs`

## `Id`

```sql
CREATE TABLE [dbo].[Genre]
(
    [Id] INT NOT NULL,
    ... ,
    CONSTRAINT PK_Genre PRIMARY KEY CLUSTERED ([Id] ASC )
);
```

Et si on veut un `auto-incrément` :
```sql
[Id] INT IDENTITY(1, 1) NOT NULL,
```

`IDENTITY(seed, increment)` par exemple si on a `IDENTITY(20,5)` on aura les `Id ` :
`20`, `25`, `30`, `35`, ...



> ## `CLUSTERED`
>
> ### 🟦 1. Qu’est-ce qu’un *clustered index* ?
>
> Un **clustered index** (index *clusterisé* en français) définit **l’ordre physique des données** dans la table.
>
> 👉 **La table elle-même est stockée physiquement triée selon les colonnes de cet index.**
>
> C’est donc *la structure de stockage principale* de la table.
>
> Dans ton exemple :
>
> ```
> PRIMARY KEY CLUSTERED ([Id] ASC)
> ```
>
> ➡️ Cela signifie :
>
> > La table *Genre* est physiquement triée par `Id` dans l’ordre croissant.
>
> ------
>
> ### 🟧 2. Conséquences concrètes
>
> ### ✔ Le `clustered index` = la table
>
> SQL Server stocke les lignes dans l’ordre du clustered index.
>  Donc :
>
> - La table n’est **pas** stockée “en vrac”
> - Son **ordre physique** = l’ordre du clustered index
>
> 📌 On dit alors que la table est un **clustered index** (IL N’Y A PAS de table “heap”).
>
> ------
>
> ### 🟩 3. Pourquoi c’est important ?
>
> #### ✔ 1. Les recherches par la clé du `clustered index` sont ultra rapides
>
> Exemple :
>
> ```sql
> SELECT * FROM Genre WHERE Id = 5;
> ```
>
> → Très performant.
>
> #### ✔ 2. Les recherches par intervalles sont performantes
>
> Exemple :
>
> ```sql
> WHERE Id BETWEEN 10 AND 100
> ```
>
> → Parcours séquentiel de données contiguës.
>
> #### ✔ 3. Le `clustered` influence tous les autres index
>
> Les index non clusterisés contiennent automatiquement la clé clusterisée.
>
> ------
>
> ### 🟥 4. Inconvénients
>
> #### ❌ 1. Les insertions “hors ordre” coûtent plus cher
>
> Si ton `clustered` est sur `Id INT` et que tu insères :
>
> - 1
> - 2
> - 3
> - 500
> - 4
>
> SQL Server doit réorganiser la table → fragmentation.
>
> C’est pourquoi :
>
> - On évite un `clustered` sur un `uniqueidentifier` aléatoire (`NEWID()`)
> - On préfère `NEWSEQUENTIALID()` si la PK est un GUID
>
> #### ❌ 2. Tu ne peux avoir **qu’un seul** `clustered index` par table
>
> Car il définit l’ordre physique unique.
>
> ------
>
> ### 🟦 5. Un `clustered index` ≠ un `primary key`
>
> Mais par défaut SQL Server :
>
> - Crée la `**Primary Key**` comme **`clustered`**
> - Sauf si tu lui dis explicitement le contraire
>
> Tu pourrais avoir :
>
> ```sql
> PRIMARY KEY NONCLUSTERED (Id)
> CLUSTERED INDEX ... (autre colonne)
> ```
>
> ------
>
> ### 🟨 6. Et si tu ne mets PAS de `clustered index` ?
>
> La table devient un **`heap`**.
>  C’est une table sans ordre physique.
>
> ✔ Utile pour certaines tables temporaires
>  ❌ À éviter pour les tables métier
>
> ------
>
> ### 🟢 Résumé court
>
> | Concept                              | Signification                                       |
> | ------------------------------------ | --------------------------------------------------- |
> | **Clustered index**                  | Détermine l’ordre physique des lignes dans la table |
> | **PRIMARY KEY CLUSTERED**            | La PK définit l’ordre physique                      |
> | **1 seul par table**                 | Un seul `clustered` est possible                    |
> | **Rapide pour recherches et plages** | Excellent pour `reads`                              |
> | **Fragile aux inserts désordonnés**  | Peut fragmenter                                     |
>
> 



## `GUID` : `UNIQUEIDENTIFIER`

```sql
CREATE TABLE [dbo].[Invitation] (
    ... ,
    [Token] UNIQUEIDENTIFIER NOT NULL DEFAULT NEWSEQUENTIALID(),
```





## `BOOLEAN` : `BIT`

Le type `boolean` est représenté dans `sql server` par le type `bit`.

Ce type prend trois valeurs `0`, `1` ou `null`.



## `Email` : `VARCHAR(254)`

## Les dates : `datetime2(0)`

> `datetime2(0)` signifie que tu utilises le type **`datetime2` avec une précision de 0 décimales pour les fractions de seconde**.
>
> Autrement dit :
>
> # 🕒 **`datetime2(0)` = date + heure, mais sans millisecondes**
>
> ### Exemple :
>
> - `2025-11-19 14:32:12` → OK
> - `2025-11-19 14:32:12.4587923` → ❌ sera arrondi à `2025-11-19 14:32:12`
>
> ------
>
> # 📌 Détails techniques
>
> `datetime2(p)` accepte un paramètre `p` de **0 à 7**, où `p` = précision des fractions de seconde :
>
> | Type           | Précision fraction de seconde | Exemple                       |
> | -------------- | ----------------------------- | ----------------------------- |
> | `datetime2(0)` | 0 digits                      | `2025-11-19 14:32:12`         |
> | `datetime2(3)` | millisecondes (ms)            | `2025-11-19 14:32:12.458`     |
> | `datetime2(7)` | ~100ns (max)                  | `2025-11-19 14:32:12.4587923` |
>
> Par défaut, SQL Server utilise **`datetime2(7)`**.
>
> ------
>
> # 🧠 Pourquoi utiliser `datetime2(0)` ?
>
> Tu choisis `0` si :
>
> - Tu n’as **pas besoin** des millisecondes ou microsecondes
> - Tu veux **réduire la taille** stockée
>
> ### Taille en octets selon la précision :
>
> | Précision `p` | Taille   |
> | ------------- | -------- |
> | 0–2           | 6 octets |
> | 3–4           | 7 octets |
> | 5             |          |

## Champ date : `DATETIME2`

```sql
[CreationLe] DATETIME2(0) DEFAULT SYSDATETIME(),
```

`DATETIME2(n)` : `n` est une valeur comprise entre `0` et `7` qui correspond à la précision après la seconde.

> Dans `DATETIME2(n)` (ou `TIME(n)`), le **n** correspond à la **précision des fractions de seconde**, c’est-à-dire **le nombre de chiffres décimaux pour les millisecondes**.
>
> # Valeur de `n` = précision des fractions de secondes
>
> | n     | Précision            | Exemple de valeur possible    |
> | ----- | -------------------- | ----------------------------- |
> | **0** | secondes entières    | `2025-01-15 10:23:45`         |
> | **1** | 0.1 seconde (100 ms) | `2025-01-15 10:23:45.1`       |
> | **2** | 0.01 seconde (10 ms) | `2025-01-15 10:23:45.12`      |
> | **3** | 1 milliseconde       | `2025-01-15 10:23:45.123`     |
> | **4** | 0.1 ms               | `2025-01-15 10:23:45.1234`    |
> | **5** | 0.01 ms              | `2025-01-15 10:23:45.12345`   |
> | **6** | 1 µs (microseconde)  | `2025-01-15 10:23:45.123456`  |
> | **7** | ~100 ns              | `2025-01-15 10:23:45.1234567` |

























