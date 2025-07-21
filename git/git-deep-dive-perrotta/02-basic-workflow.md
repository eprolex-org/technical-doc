# 02 Basic `workflow`

## Les `2` questions

### Comment cette `command` déplace les informations entre les `4 zones` ?

### Comment cette `command` modifie le `Repository` ?



## Modification d'un fichier

<img src="assets/modify-file-yyhsnjuiekjhhqqa.png" alt="modify-file-yyhsnjuiekjhhqqa" />

`menu.txt` n'ets plus alligné avec `Index` et `Repository`.

<img src="assets/git-status-yyeeahyytrzyyhgf.png" alt="git-status-yyeeahyytrzyyhgf" />

### `git status` nous conseille d'effectuer un `git add`, un `git restore` ou bien un `git commit -a`.



## Déplacer les données vers la droite

<img src="assets/git-add-ggscxxxvbwgqtttaj.png" alt="git-add-ggscxxxvbwgqtttaj" />

`git add` ajoute le fichier à l'`Index` :

<img src="assets/file-is-staged-rrqvcfadsezrmlknggggg.png" alt="file-is-staged-rrqvcfadsezrmlknggggg" />

`menu.txt` est `staged`, `indexé`.

Si j'execute un `git diff`, j'obtiens une réponse vide.

Par contre si j'exécute la commande `git diff --cached` :

<img src="assets/git-diff-cached-option-yyhgfsdcevtwnjksiuyetra.png" alt="git-diff-cached-option-yyhgfsdcevtwnjksiuyetra" style="zoom:25%;" />

En résumé j'ai la situation :

<img src="assets/summary-git-diff-pplfdrteghsbvcxwwauijhs.png" alt="summary-git-diff-pplfdrteghsbvcxwwauijhs" />

Maintenant si je fais un `commit`, j'aligne `menu.txt` dans le `Repository`.

`commit` ne se contente pas d'aligner le contenu, il créé un nouveau `commit` et  il déplace la `branch`. `commit` modifie le `Repository`.

On reviens à un `clean status` et `git diff` ou bien `git diff --cached` ne montre rien.

<img src="assets/clean-status-ksqvfxrtzunjkia.png" alt="clean-status-ksqvfxrtzunjkia" />



## Déplacer les données vers la gauche

### la commande `switch`

Elle permet de passer d'une `branch` à l'autre.

> `switch ` ne permet de passer que d'une `branch` à une autre, alors que `checkout` permet de se rebdre sur n'importe quel `commit`.

Elle fait deux choses :

1. Dans le `Repository`, elle déplace la réferece  `HEAD`, ce qui change le `current commit`.
2. Elle copie les données dans `Working Area` et `Index`.

<img src="assets/git-switch-command-gdfretbvcgdtrefcvdghytrefdcsx.png" alt="git-switch-command-gdfretbvcgdtrefcvdghytrefdcsx" />

Le `HEAD` détermine le `current commit`.

<img src="assets/head-file-text-pmloiakjshvaaqszeartx.png" alt="head-file-text-pmloiakjshvaaqszeartx" />







