# 07 Gestion des `line separator` : `CRLF` et `LF`



## ✅ Gestion des fins de ligne (LF) dans le projet

Pour garantir des fins de ligne cohérentes (`LF`) sur toutes les plateformes (Windows, macOS, Linux) et éviter les faux changements dans Git :

---

### 1. `.gitattributes` (à placer à la racine du dépôt)

```gitattributes
* text=auto

# Fichiers texte forcés en LF
*.cs       text eol=lf
*.csproj   text eol=lf
*.sln      text eol=lf
*.json     text eol=lf
*.xml      text eol=lf
*.razor    text eol=lf
*.config   text eol=lf

# Fichiers binaires
*.png      binary
*.dll      binary
```

---

### 2. `.editorconfig` minimaliste (à placer à la racine)

```ini
root = true

[*]
end_of_line = lf
```

---

### 3. Configuration Git locale (une seule fois par développeur)

#### Pour macOS / Linux :

```bash
git config --global core.autocrlf input
```

#### Pour Windows :

```bash
git config --global core.autocrlf true
```

---

### 4. (Optionnel) Normalisation des fichiers existants

À exécuter une seule fois après ajout du `.gitattributes` :

```bash
git rm --cached -r .
git reset --hard
git add .
git commit -m "Normalisation des fins de ligne"
```

---

✅ **Résultat :**  

- Fins de ligne normalisées (`LF`) dans Git.  
- Comportement cohérent sur tous les systèmes et éditeurs.  
- Plus de faux changements dus aux fins de ligne dans `git diff`, `Rider`, `VS`, `Sourcetree` ou `Azure DevOps`.

### Comparaison avec d'autres valeurs

| **Valeur**            | **Description**                                              |
| --------------------- | ------------------------------------------------------------ |
| `core.autocrlf=true`  | Convertit **LF → CRLF** lors du checkout et **CRLF → LF** lors du commit. Utilisé sur Windows. |
| `core.autocrlf=false` | Désactive toutes les conversions de fins de ligne. Les fichiers restent tels quels. |
| `core.autocrlf=input` | Convertit **CRLF → LF** lors du commit, mais ne modifie pas les fichiers lors du checkout. |
