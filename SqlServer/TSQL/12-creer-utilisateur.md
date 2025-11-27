# Créer un utilisateur 

## Créer un `login`

Pour la `Connection String`,  `User ID` correspond à `login`.

> SQL Server a **deux niveaux d’identité** : un *login* (serveur) et un *user* (base). La `connection string`, elle, ne connaît que le **login**.

En `SQL` ça se traduit par :
```sql
CREATE LOGIN MonAppLogin WITH PASSWORD='...';
```



## Créer un `user`

Un `User` permet de définir des permissions, le `login` sert à se connecter.
```sql
CREATE USER MonAppUser FOR LOGIN MonAppLogin;
```

> ## 🖼️ Résumé clair et court
>
> | Concept   | Niveau  | Rôle                      | Correspond à                        |
> | --------- | ------- | ------------------------- | ----------------------------------- |
> | **Login** | Serveur | S’authentifier            | `User ID` dans la connection string |
> | **User**  | Base    | Permissions dans une base | `CREATE USER ... FOR LOGIN`         |
>
> Le login ⇒ se connecte au serveur
>  Le user ⇒ définit ce qu’il peut faire dans la base



## Ajouter des droits

```sql
USE MaBase;

GRANT SELECT ON SCHEMA::dbo TO MonAppUser;
GRANT INSERT ON SCHEMA::dbo TO MonAppUser;
GRANT UPDATE ON SCHEMA::dbo TO MonAppUser;

-- Permet d'exécuter les fonctions / procédures
GRANT EXECUTE ON SCHEMA::dbo TO MonAppUser;

-- IMPORTANT : s'assurer que DELETE n'est pas permis
DENY DELETE ON SCHEMA::dbo TO MonAppUser;
```



## Script complet

```sql
------------------------------------------------------------
-- 1. Créer le LOGIN (serveur) utilisé par la connection string
------------------------------------------------------------
CREATE LOGIN AppLogin 
WITH PASSWORD = 'MotDePasseTrèsFort!';   -- À changer bien sûr


------------------------------------------------------------
-- 2. Créer l’USER (base) lié au LOGIN
------------------------------------------------------------
USE MaBase;
GO

CREATE USER AppUser FOR LOGIN AppLogin;
GO


------------------------------------------------------------
-- 3. Donner permissions SELECT / INSERT / UPDATE sur tout le schéma
--    (sans jamais donner DELETE)
------------------------------------------------------------

-- Lecture
GRANT SELECT ON SCHEMA::dbo TO AppUser;

-- Écriture sauf DELETE
GRANT INSERT ON SCHEMA::dbo TO AppUser;
GRANT UPDATE ON SCHEMA::dbo TO AppUser;

-- Interdire explicitement DELETE (par sécurité)
DENY DELETE ON SCHEMA::dbo TO AppUser;


------------------------------------------------------------
-- 4. Autoriser l’exécution des fonctions, vues, procédures
------------------------------------------------------------
GRANT EXECUTE ON SCHEMA::dbo TO AppUser;


------------------------------------------------------------
-- 5. (Facultatif) S’assurer que l’utilisateur ne peut pas modifier le schéma
------------------------------------------------------------
DENY ALTER ON SCHEMA::dbo TO AppUser;
DENY CONTROL ON SCHEMA::dbo TO AppUser;
```

> ## 📌 Notes
>
> - **`Triggers`** : fonctionnent automatiquement avec INSERT/UPDATE.
> - **`Indexes`** : aucun réglage, SQL Server les utilise tout seul.
> - **`Views`** : SELECT couvre déjà l'accès.
> - **`Functions`** : `GRANT EXECUTE` est nécessaire.
> - **Pas de `DELETE`** : assuré par `DENY DELETE`.



















