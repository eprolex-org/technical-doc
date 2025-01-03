# 07 Gestion des `line separator` : `CRLF` et `LF`

Il faut créer un fichier à la racine du dépôt : 

`.gitattributes`

```
* text eol=crlf
```

Puis appliquer la configuration

```bash
git add --renormalize .
git commit -m "Normalize line endings to CRLF"
```



## Empêcher `git` de réécrire les fins de ligne

```bash
git config --global core.autocrlf false
```

### Comparaison avec d'autres valeurs

| **Valeur**            | **Description**                                              |
| --------------------- | ------------------------------------------------------------ |
| `core.autocrlf=true`  | Convertit **LF → CRLF** lors du checkout et **CRLF → LF** lors du commit. Utilisé sur Windows. |
| `core.autocrlf=false` | Désactive toutes les conversions de fins de ligne. Les fichiers restent tels quels. |
| `core.autocrlf=input` | Convertit **CRLF → LF** lors du commit, mais ne modifie pas les fichiers lors du checkout. |
