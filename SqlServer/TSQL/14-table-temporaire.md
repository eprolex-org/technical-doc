# 14 Les tables `temporaires` : `Temp`

 

## Syntaxe

```sql
SELECT ...
INTO #new_table
FROM ...
WHERE ...
```

> ### ! à la clause `INTO` et `#`
>
> La clause `INTO` créé une table temporaire ou non.
>
> ###  Où va la table ?
>
> - `INTO #Temp` → table temporaire **locale** (session courante)
> - `INTO ##Temp` → table temporaire **globale** (visible par d’autres sessions tant qu’elle existe)
> - `INTO dbo.MaTable` → table “normale” dans le schéma indiqué
>
> Donc, si on veut une table temporaire on DOIT préfixé le nom par `#`.



## Exemple 

```sql
-- Créer un set de déposant filtrer
DECLARE @now datetime2 = SYSDATETIME()
DECLARE @currentUtilisateurId int = 10
DECLARE @recherche nvarchar(210) = N'%'+N''+N'%'


SELECT DISTINCT
    dep_u.Id AS UtilisateurId,
    dep_u.Prenom,
    dep_u.Nom,
    dep_u.Email,
    dep_u.Fixe,
    dep_u.Gsm,
    dep_u.Genre,
    dep_u.Langue
INTO #temp_dep
FROM Deposant dep
JOIN Utilisateur dep_u
ON dep.UtilisateurId = dep_u.Id
JOIN Demandeur dem
ON dep.DemandeurId = dem.Id
JOIN Utilisateur dem_u
    ON dem.UtilisateurId = dem_u.Id
JOIN Utilisateur current_u
ON current_u.Id = @currentUtilisateurId
WHERE
    -- Vérifier que les utilisateur n'ont pas été supprimés
dep_u.EstSupprime = 0
AND dem_u.EstSupprime = 0
AND current_u.EstSupprime = 0
-- Vérfier que le déposant et le demandeur sont bien valide (actif)
AND dem.DebutValidite <= @now
AND (dem.FinValidite IS NULL OR dem.FinValidite > @now)

AND dep.DebutValidite <= @now
AND (dep.FinValidite IS NULL OR dep.FinValidite > @now)
-- Recherche par Nom ou Email
AND (dep_u.Nom COLLATE Latin1_General_100_CI_AI LIKE @recherche
         OR dep_u.Email COLLATE Latin1_General_100_CI_AI LIKE @recherche)
-- Recherche dans un groupe de demandeur
AND dep.DemandeurId IN (2, 3, 4)


SELECT *
FROM #temp_dep

SELECT COUNT(*)
FROM #temp_dep
```

On créé un set de données complexes et filtrées. Puis on peut en tirer les informations qu'on veut sans devoir répéter les clauses `WHERE` et autre.