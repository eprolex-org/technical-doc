# 02 Problème avec les fichiers `resx` et `designer`



## Création d'une `resource`

Je créé un dossier `Resources` et dedans j'ajoute un fichier `.resx` :

<img src="assets/resx-creation.png" alt="resx-creation" style="zoom:50%;" />

Si je clique sur le fichier `.designer.cs` je constate qu'il est vide.



## Ajout d'une ressource

Dans `Program.cs`, si j'essaye d'ajouter un texte aux `Resources` :

<img src="assets/move-to-resource.png" alt="move-to-resource" />

J'obtiens alors l'erreur suivante :

<img src="assets/error-resx-file-not-available.png" alt="error-resx-file-not-available" style="zoom:33%;" />



### Correction

On doit aller dans les settings de `Rider`

<img src="assets/setting-resx-disable.png" alt="setting-resx-disable" />

Et on `enable` `ResX Generator`

Maintenant si j'ajoute un fichier `resx`, j'obtiens bien une classe `.designer.cs`.

<img src="assets/designer-file-generated.png" alt="designer-file-generated" style="zoom:33%;" />



## Enregistrer une nouvelle `resource`

Dans les `Context Actions` j'ai maintenant directement la possibilité de transformer une chaîne en nouvelle `resource` :

<img src="assets/quick-action-move-resource.png" alt="quick-action-move-resource" style="zoom:33%;" />

<img src="assets/localization-manager-hello.png" alt="localization-manager-hello" />

Et le code modifié :

```cs
Console.WriteLine(App.HelloWorld);
```

