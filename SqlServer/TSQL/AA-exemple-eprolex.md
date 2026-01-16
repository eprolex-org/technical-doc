# Exemple Complexe dans `Eprolex`

## Liste des déposants et leurs demandeurs associés

Pour avoir une pagination, on va avoir besoin de trois requête.

On a comme critères de filtre pour la liste :

- Une `recherche` par `nom` ou `email`
- Un filtre par `demandeur`
- Un tri sur l'`ordre alphabétique`
- La `page` voulu
- Le nombre de déposant par page ici `ParPage`



### Nombre total de déposants

```sql
DECLARE @currentUserId int = 10
DECLARE @now datetime2 = SYSDATETIME()
DECLARE @recherche nvarchar(210) = N'%' + N'' + N'%'


SELECT COUNT(DISTINCT u.Id)
FROM Deposant dep
JOIN Utilisateur u
ON u.Id = dep.UtilisateurId
JOIN Demandeur dem
ON dem.Id = dep.DemandeurId
JOIN Utilisateur du
ON du.Id = dem.UtilisateurId
JOIN Utilisateur cu
ON cu.Id = @currentUserId
WHERE cu.EstSupprime = 0
AND du.EstSupprime = 0
AND u.EstSupprime = 0

AND dep.DebutValidite <= @now
AND (dep.FinValidite IS NULL OR dep.FinValidite > @now)

AND dem.DebutValidite <= @now
AND (dem.FinValidite IS NULL OR dem.FinValidite > @now)

AND dep.DemandeurId IN (2, 3, 4)

AND (u.Nom COLLATE Latin1_General_100_CI_AI LIKE @recherche
           OR u.Email COLLATE Latin1_General_100_CI_AI LIKE @recherche)


```





### La liste des utilisateurs déposants (`Ids` seulement) paginée

```sql

DECLARE @currentUserId int = 10
DECLARE @now datetime2 = SYSDATETIME()
DECLARE @recherche nvarchar(210) = N'%' + N'' + N'%'

DECLARE @parPage int = 2;
DECLARE @page int = 3;

-- Sécurise (évite offset négatif / fetch invalide)
DECLARE @offset int = (CASE WHEN @page < 1 THEN 0 ELSE (@page - 1) * @parPage END);

SELECT DISTINCT u.Id
FROM Deposant dep
         JOIN Utilisateur u
              ON u.Id = dep.UtilisateurId
         JOIN Demandeur dem
              ON dem.Id = dep.DemandeurId
         JOIN Utilisateur du
              ON du.Id = dem.UtilisateurId
         JOIN Utilisateur cu
              ON cu.Id = @currentUserId
WHERE cu.EstSupprime = 0
  AND du.EstSupprime = 0
  AND u.EstSupprime = 0

  AND dep.DebutValidite <= @now
  AND (dep.FinValidite IS NULL OR dep.FinValidite > @now)

  AND dem.DebutValidite <= @now
  AND (dem.FinValidite IS NULL OR dem.FinValidite > @now)

  AND dep.DemandeurId IN (2, 3, 4)
  
  AND (u.Nom COLLATE Latin1_General_100_CI_AI LIKE @recherche
           OR u.Email COLLATE Latin1_General_100_CI_AI LIKE @recherche)
           
ORDER BY u.Id
    OFFSET @offset ROWS
FETCH NEXT @parPage ROWS ONLY;
```





### La liste des déposants et de leurs demandeurs

```sql

DECLARE @currentUserId int = 10
DECLARE @now datetime2 = SYSDATETIME()
DECLARE @recherche nvarchar(210) = N'%' + N'' + N'%'



SELECT
    u.Id,
    u.Prenom,
    u.Nom,
    u.Email,
    u.Fixe,
    U.Gsm,
    dem.Id,
    du.Id AS UtilisateurId,
    du.Prenom,
    du.Nom,
    du.Genre,
    dep.EstActif
FROM Deposant dep
JOIN Utilisateur u
ON dep.UtilisateurId = u.Id
JOIN Demandeur dem
ON dep.DemandeurId = dem.Id
JOIN Utilisateur du
ON dem.UtilisateurId = du.Id
JOIN Utilisateur cu
ON cu.Id = @currentUserId
WHERE u.EstSupprime = 0
  AND du.EstSupprime = 0
  AND cu.EstSupprime = 0

   AND dep.DebutValidite <= @now
  AND (dep.FinValidite IS NULL OR dep.FinValidite > @now)

  AND dem.DebutValidite <= @now
  AND (dem.FinValidite IS NULL OR dem.FinValidite > @now)
  
  AND u.Id IN (6, 7)

  AND dep.DemandeurId IN (2, 3, 4)

  AND (u.Nom COLLATE Latin1_General_100_CI_AI LIKE @recherche
           OR u.Email COLLATE Latin1_General_100_CI_AI LIKE @recherche)
           
ORDER BY u.Nom, u.Prenom, du.Nom, du.Prenom

```

