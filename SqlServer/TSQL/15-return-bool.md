#  15. Retourner un booléen - Faire un test

Le mieux est de mixer un `CAST` avec un `CASE` :

```sql
DECLARE @now datetime2(7) = SYSDATETIME();

SELECT CAST(
    CASE 
    	WHEN EXISTS (
            SELECT 1
            FROM dbo.Invitation
            WHERE Token = @token
              AND Statut = 1
              AND UtiliseLe IS NULL
              AND DebutValidite <= @now
              AND FinValidite  >  @now
            ) THEN 1 
        ELSE 0 
    END
AS bit) AS IsValid;
```



## Syntaxe générale du `CASE`

```sql
CASE
	WHEN boolean_condition1 THEN result1
	WHEN boolean_condition2 THEN result2
	ELSE result3
END
```

