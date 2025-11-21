# 00. Syntaxe `TSQL`

## `;` et `GO`

### 🟦 1. `;` = **fin d’instruction T-SQL**

Le point-virgule est le **vrai terminateur d’instruction** dans T-SQL (comme `;` en C#).

Exemple :

```sql
SELECT * FROM Users;
UPDATE Users SET IsActive = 1 WHERE Id = @id;
```

### Aujourd’hui :

- Il était **optionnel** pendant longtemps.
- Microsoft souhaite le rendre **obligatoire** dans les versions futures.

### 🟧 2. `GO` = **directive pour le client SQL (SSMS, sqlcmd)**

⚠️ Ce n’est **pas** du T-SQL.

`GO` sert à dire à SSMS / sqlcmd / Azure Data Studio :

> « Envoie ce bloc d’instructions au serveur comme un batch séparé ».

Exemple :

```sql
CREATE TABLE X (...);
GO

INSERT INTO X VALUES (...);
GO
```

🎯 Cela signifie :

- Premier batch : créer la table
- Deuxième batch : insérer les données



## Qu'est-qu'un `barch`

Voici une explication **claire, précise et pratique** de ce qu'est un **batch**, en T-SQL et de manière générale :

------

### 🟦 1. Définition générale

Un **batch** est un **groupe d'instructions SQL envoyé d’un seul coup au moteur SQL** pour qu’il l’analyse, le compile (si nécessaire) et l’exécute d’un bloc.

C’est une *unité logique d’exécution*.

En d'autres termes :

> **Un batch = un paquet de commandes SQL que le serveur traite ensemble.**
>  **Il est envoyé au serveur en une seule fois.**

------

### 🟧 2. Dans SQL Server (T-SQL)

#### ✔ Comment crée-t-on un batch ?

Dans SQL Server, un batch est généralement délimité par :

- **`GO`**, mais *attention* :
   **`GO` n’est pas du T-SQL** → c'est une commande pour SSMS / sqlcmd.
   Elle indique : “envoie ce qu’il y a avant comme un batch séparé”.

Exemple :

```sql
SELECT 1;
SELECT 2;
GO      -- Fin du batch 1

SELECT 3;
GO      -- Fin du batch 2
```

------

### 🔍 3. Pourquoi les `batches` sont importants ?

Parce qu'ils influencent **la portée**, **la compilation** et **la validité** de certaines instructions.

------

### 🎯 3.1 portée des variables (scope)

Les variables déclarées dans un batch **n’existent pas dans le batch suivant**.

❌ Mauvais :

```sql
DECLARE @x int = 5;
GO

SELECT @x;  -- Erreur : @x n'existe plus
```

------

### 🎯 3.2 Certaines instructions doivent être **en début de batch**

Exemples :

- `CREATE PROCEDURE`
- `CREATE VIEW`
- `CREATE TRIGGER`
- `CREATE FUNCTION`

Tu ne peux pas écrire ceci :

```sql
DECLARE @i int;
CREATE PROCEDURE P AS SELECT @i;
```

→ Erreur : une instruction `CREATE PROCEDURE` doit être le **premier** mot du batch.

Il faut séparer :

```sql
DECLARE @i int;
GO

CREATE PROCEDURE P AS SELECT 1;
GO
```

------

### 🎯 3.3 Recompilation et plan d’exécution

Chaque batch peut :

- déclencher une compilation separate
- produire un plan d’exécution distinct
- réutiliser ou non un plan précédent

Cela joue sur les performances.

------

### 🎯 3.4 Transactions (à ne pas confondre !)

👉 Un **batch ≠ une transaction**.

Tu peux avoir :

- plusieurs `transactions` dans un `batch`
- un `batch` sans `transaction`
- une `transaction` qui s'étend sur plusieurs `batches` (mais ce n'est pas recommandé en scripts)

Exemple valide :

```sql
BEGIN TRAN;
UPDATE Users SET IsActive = 1;
GO       -- batch séparé
UPDATE Users SET Logins = Logins + 1;
COMMIT;
GO
```

## 🧠 Résumé clair

| Concept          | Signification                                               |
| ---------------- | ----------------------------------------------------------- |
| **Batch**        | Ensemble d’instructions SQL envoyées ensemble au moteur SQL |
| **Délimitation** | Avec `GO` (directive client, pas T-SQL)                     |
| **Scope**        | Les variables existent uniquement dans leur batch           |
| **Contraintes**  | Certaines commandes `CREATE` doivent commencer un batch     |
| **Compilation**  | Le serveur compile chaque batch comme une unité             |



## `[dbo].[Utilisateur]` ou `[Utilisateur]` ou `Utilisateur` ?

### 🟩 Résumé professionnel

| Situation                           | Recommandation                                 |
| ----------------------------------- | ---------------------------------------------- |
| Nommer une table                    | `[dbo].[NomTable]`                             |
| Nommer une colonne                  | `[NomColonne]`                                 |
| Accès complet                       | `[dbo].[NomTable].[NomColonne]`                |
| Avec alias                          | `U.[NomColonne]`                               |
| Scripting, migrations, déploiements | Toujours utiliser `[]` et qualifier par schéma |

### 📌 Quand faut-il mettre des `[]` ?

Toujours si :

- le nom contient un espace → `[User Info]`
- le nom contient un point → `[Nom.Table]`
- le nom est réservé → `[User]`, `[Order]`, `[Group]`
- tu veux être 100% safe dans les scripts d’`install`

Si tu choisis des noms “propres” (pas de mots réservés, pas d’espaces), tu pourrais techniquement t’en passer :

```sql
SELECT Id FROM dbo.Utilisateur;
```

… mais dans les scripts SQL professionnels, on met **toujours** `[]`.



##  Que veut dire `[dbo]` ?

`dbo` signifie **`DataBase Owner`**.

C’est **le schéma par défaut** créé pour chaque base SQL Server.

En résumé :

- `[dbo]` est un **schéma** (un `namespace` dans la base).
- Une table s’appelle donc vraiment :
   👉 `[dbo].[Utilisateur]`

Par défaut, quand tu crées une table sans préciser le schéma, elle va dans `dbo`.



## Qu'est-ce qu'un schéma dans SQL Server ?

Un schéma est une **catégorie logique** qui contient tables, vues, fonctions, etc.

Cela sert à :

- Organiser les objets (comme des `namespaces` ou des dossiers)
- Gérer les permissions par groupe
- Éviter les collisions : deux schémas peuvent avoir une table du même nom



## Est-ce une bonne idée d’utiliser un schéma `[enum]` pour les tables représentant des énumérations `C#` ?

👉 **Oui, c’est une EXCELLENTE idée**, et de nombreuses équipes le font.

### ✔ Avantages :

#### 1. **Organisation claire**

Tu sépares les tables "techniques" ou "de référence" du reste :

```sql
enum.Statut
enum.TypeDocument
enum.Genre
```

Au lieu de polluer `dbo`.

#### 2. **Lisibilité accrue dans la DB**

Quand tu listes les tables, tu identifies immédiatement les “tables `enum`”.

#### 3. **Permissions faciles à gérer**

Tu peux donner des droits en lecture seule à ce schéma pour certaines applis :

```sql
GRANT SELECT ON SCHEMA::enum TO AppUser;
```

#### 4. **Évite les collisions**

Tu peux avoir :

- `[dbo].[StatutTask]`
- `[enum].[Statut]`

…sans confusion.

#### 5. **Parfait pour les tables de référence “stables”**

Les `enums` métier ne changent presque jamais.













