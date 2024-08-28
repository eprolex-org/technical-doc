#  01 Les objets de `Git`

## `Porcelain` commands

```bash
git add
git commit
git push
git pull
git branch
git switch # (git checkout)
git merge
git rebase
```

`git switch` est une version récente remplaçant pour certaine fonction l'ancien `git checkout`. `git checkout` continu de fonctionner.

## `Plumbing` commends (`Plomberie`)

Les briques composants les `Porcelain` commands.

```bash
git cat-file
git hash-object
git count-objects
```



## La base de `Git`

### C'est un stupide `tracker` de contenu

### git - the stupid content tracker

```bash
man git

GIT(1)                            Git Manual                            GIT(1)

NAME
       git - the stupid content tracker
```

C'est un `map` persistant sur le disque, une table associative utilisant un `hash` comme clé.





## Les objets

<img src="assets/four-object-in-git-database.png" alt="four-object-in-git-database" />

Il ya quatres sortes d'objets :

1. Les `blobs` (le contenu d'un fichier compressé)
2. Les `trees` (des sortes de répertoires)
3. Les `commits`
4. Les `tags`
