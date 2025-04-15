# CC `FileInfo` vs `File`

(ChatGPT)

### Pourquoi deux classes : `File` (statique) et `FileInfo` (à instancier) ?

**1. `File` (statique)**

- C’est une classe **utilitaire**.
- Fournit des **méthodes statiques** rapides pour des opérations de base sur les fichiers : lire, écrire, copier, supprimer, etc.
- Ne conserve **aucun état** sur le fichier : chaque appel est indépendant.
- Idéal pour des scripts ou des manipulations ponctuelles.

Exemples :

```ruby
File.Copy("source.txt", "dest.txt");
File.Delete("temp.txt");
string text = File.ReadAllText("data.txt");
```

------

**2. `FileInfo` (objet à instancier)**

- Représente **un fichier en tant qu’objet** avec des **propriétés** (comme `Length`, `CreationTime`, etc.).
- Permet une approche **orientée objet**.
- Peut être plus efficace si tu fais **plusieurs opérations** sur le même fichier, car certaines infos sont **mises en cache** (tu évites certains accès disque).
- S'intègre mieux avec d'autres patterns OOP.

Exemples :

```ruby
var file = new FileInfo("data.txt");

if (file.Exists)
{
    Console.WriteLine(file.Length);
    file.CopyTo("backup.txt");
}
```

------

### Faut-il privilégier l'une ou l'autre ?

#### Utilise `File` quand :

- Tu fais des **opérations rapides** et indépendantes.
- Tu veux un code simple et court.
- Tu n’as pas besoin de lire plusieurs fois des métadonnées (taille, dates, etc.).

#### Utilise `FileInfo` quand :

- Tu fais **plusieurs opérations** sur le **même fichier**.
- Tu veux accéder à des **informations** du fichier (longueur, date de création…).
- Tu préfères un style **orienté objet**.

------

### En résumé :



| Cas d’usage                              | Classe recommandée |
| ---------------------------------------- | ------------------ |
| Opération unique, rapide                 | `File`             |
| Plusieurs opérations ou infos du fichier | `FileInfo`         |
| Style orienté objet                      | `FileInfo`         |
| Style fonctionnel/procédural             | `File`             |

Tu peux appliquer exactement la même logique pour `Directory` vs `DirectoryInfo`.