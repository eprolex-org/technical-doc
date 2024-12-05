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

