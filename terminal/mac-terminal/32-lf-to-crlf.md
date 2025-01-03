# 32 Passer de `LF` à `CLRF`

## Problématique

Un projet `.net` créé sur Mac peut-être par défaut avec un `line separator` de type `LF`. Même si on règle l'`IDE` pour être par défaut en `CRLF`, les anciens fichiers restent eux en `LF`. Il faut donc trouver un moyen de passer tous les fichiers d'un projet à `CRLF` ou vice versa suivant les besoins.



## Convertir les fichiers

### 1. **Naviguez jusqu'à la racine de votre projet :**

   ```bash
   cd /chemin/vers/votre/projet
   ```



### 2. **Convertir tous les fichiers dans le projet :** Exécutez cette commande pour trouver tous les fichiers et les convertir en **CRLF** :

   ```bash
  find . -type f -exec unix2dos {} \;
   ```

   - **`find .`** : Rechercher tous les fichiers dans le répertoire actuel et ses sous-répertoires.
   - **`-type f`** : Filtrer pour ne sélectionner que les fichiers.
   - **`-exec unix2dos {}`** : Appliquer la commande `unix2dos` à chaque fichier trouvé.



### 3. Exclure certain fichier

```bash
find . -type f -not -name "*.png" -not -name "*.jpg" -exec unix2dos {} \;
```



### 4. Exécuter sur un fichier

```bash
unix2dos chemin/vers/fichier
```



### 5. Contrôle de l'opération

```bash
find . -type f -exec file {} \; | grep 'with LF'

# ou

find . -type f -exec file {} \; | grep 'with CRLF'
```

