# 02 Les `Types` et les `Champs`



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