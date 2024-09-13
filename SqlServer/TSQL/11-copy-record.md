# 11 Copier un `enregistrement`

```sql
DECLARE @cols NVARCHAR(MAX);

-- Générer la liste des colonnes sauf 'ClientID'
SELECT @cols = STRING_AGG(COLUMN_NAME, ', ')
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'DemandeAvis'
AND COLUMN_NAME != 'Id';
-- On obtient : 'DemandeurId, TitreCode, NumeroRole, NumeroAttente, StatutCode, EstConjointe, Reference, NormeCode, IntituleFr, IntituleNl, IntituleDe, DelaiCode, DelaiMotivation, Remarques, JustificationRenvoi, DepotLe, DepotPar, CreationLe, CreationPar, ModificationLe, ModificationPar, SuppressionLe, SuppressionPar, MethodeValidationCode' 

-- Requête dynamique pour insérer les colonnes
DECLARE @sql NVARCHAR(MAX);
SET @sql = 'INSERT INTO DemandeAvis (' + @cols + ')
            SELECT ' + @cols + '
            FROM DemandeAvis
            WHERE Id = 17;';

-- Exécuter la requête
EXEC sp_executesql @sql;
```

