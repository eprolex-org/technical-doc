# 04. Supprimer et renommer des `fichiers`

## Supprimers des fichiers `git rm`

On va  d'abord créer un fichier :

<img src="assets/new-file-git-status-kkjhsyterfdgsssczdeqr.png" alt="new-file-git-status-kkjhsyterfdgsssczdeqr" />

Le fichier se trouve uniquement dans le `Working area`, il n'est pas encore indexé.

<img src="assets/new-file-in-working-area-ddsfretsygqbvvczdserfqgz.png" alt="new-file-in-working-area-ddsfretsygqbvvczdserfqgz" />

On doit maintenant faire un `git add` pour l'ajouter à l'`Index` :

<img src="assets/new-file-git-add-eeeeertsgsbvxcdsfsgssghqy.png" alt="new-file-git-add-eeeeertsgsbvxcdsfsgssghqy" />

Nous avons donc la situation :

<img src="assets/new-file-is-added-index-iiiokjhdgteyusiao.png" alt="new-file-is-added-index-iiiokjhdgteyusiao" />

### Que faire si l'on veut retirer `copyright.txt` de l'`Index` ? 

### `git rm copyright.txt`

<img src="assets/new-file-git-rm-ooaasqezdzfrxtaaszqedzfq.png" alt="new-file-git-rm-ooaasqezdzfrxtaaszqedzfq" />

On obtient une erreur car il y a deux options :

`--cached` qui ne supprime pas le fichier mais le retire de l'index

<img src="assets/git-rm-cached-pplopploiuegdfdtrsyuzjqnbvxcfdgs.png" alt="git-rm-cached-pplopploiuegdfdtrsyuzjqnbvxcfdgs" />

`-f` la version destructive qui supprime le fichier

<img src="assets/rm-distructive-kkjuyhsdefvwxwxwxwxwcqyu.png" alt="rm-distructive-kkjuyhsdefvwxwxwxwxwcqyu" />

### La commande opposée à `git add` est `git rm --cached`.



## Renommer des fichiers

Si je renomme un fichier `menu.txt` en `menu.md` :

```bash
mv menu.txt menu.md
```

<img src="assets/new-rebamming-file-polkdserztgqaszeqrdzftsyuqix.png" alt="new-rebamming-file-polkdserztgqaszeqrdzftsyuqix" />

Pour `Git` c'est l'équivalent à la suppression de `menu.txt` et l'ajout de `menu.md`.

<img src="assets/three-area-after-renamming-hgdtyueijkqioopamncvfdrts.png" alt="three-area-after-renamming-hgdtyueijkqioopamncvfdrts" />

J'ai donc la situation ci-dessus.

Je dois ajouter `menu.md` à l'`index`, car il est `untracked` (non suivis).

<img src="assets/adding-renamming-file-to-index-oolkjhgfdsrterdfgfdfcvsgxbwqqa.png" alt="adding-renamming-file-to-index-oolkjhgfdsrterdfgfdfcvsgxbwqqa" />

On a maintenant une situation comme ceci :

<img src="assets/state-after-adding-renamming-file-oopsfrezsqdar.png" alt="state-after-adding-renamming-file-oopsfrezsqdar" />

Pour renseigner la suppression de `menu.txt`, il faut maintenant `git add` le fichier `menu.txt` qui est supprimée, et cette suppression va écraser le fichier `menu.txt` de l'`Index`. `Git` met à jour le fichier `menu.txt` de l'`Index` avec celui du `Working area`, c'est à dire avec `rien`.

```bash
git add menu.txt
```

<img src="assets/index-updated-with-renamming-hhgdfretyuaanbxvcsdeaq.png" alt="index-updated-with-renamming-hhgdfretyuaanbxvcsdeaq" />

`Git` comprends alors que c'est un renommage de `menu.txt` en `menu.md`.

<img src="assets/git-status-renamming-comprehension-llksretfdgstrayquieo.png" alt="git-status-renamming-comprehension-llksretfdgstrayquieo" />

> ## `git mv`
>
> On peut aussi utiliser `git mv` :
>
> ```bash
> git mv menu.md menu.txt
> ```
>
> <img src="assets/git-renamming-with-mv-hhsytrefdgyuuiaoplkjhd.png" alt="git-renamming-with-mv-hhsytrefdgyuuiaoplkjhd" />
>
> Cette commande est peut utilisée.





