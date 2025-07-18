# 08 Git `flow`

> Conversation avec Chat GPT

### Si on essaye de pratiquer le git flow dans une petite équipe de développement, quand doit-on utiliser rebase et quand doit-on utiliser merge ?



###### ChatGPT a dit :

Très bonne question ! Lorsque tu pratiques **Git Flow** dans une petite équipe, il est important de bien utiliser `merge` et `rebase` pour garder un historique clair, tout en évitant de perturber le travail des autres développeurs.

Voici un résumé des **principes** à suivre :

------

### 🔁 **Quand utiliser `rebase` ?**

Utilise `rebase` pour **garder un historique linéaire et propre**, mais uniquement **sur des branches locales et non partagées**.

#### 👉 Typiquement :

- Avant de pousser une **feature branch** vers `origin`, pour la mettre à jour avec la dernière version de `develop` :

  ```bash
  git checkout ma-feature
  git fetch origin
  git rebase origin/develop
  ```

- Quand tu fais un **pull** sur une branche locale pour la mettre à jour sans polluer l’historique :

  ```bash
  git pull --rebase
  ```

#### ⚠️ À éviter :

- **Ne jamais rebaser une branche partagée** (déjà poussée sur `origin` et utilisée par d'autres). Cela peut créer des conflits inutiles et rendre la collaboration pénible.

------

### 🔀 **Quand utiliser `merge` ?**

Utilise `merge` pour **rassembler des branches et conserver l’historique des intégrations**.

#### 👉 Typiquement :

- Pour **fusionner une feature branch dans `develop`** (via pull request ou merge direct).
- Pour **fusionner `develop` dans une release branch**, ou inversement.
- Pour **fusionner une release dans `main`** et ensuite dans `develop` (comme le veut Git Flow).
- Pour **garder une trace des intégrations** dans les branches principales (`develop`, `main`).

```bash
git checkout develop
git merge ma-feature
```

------

### 🔧 En résumé :

| Action                                  | Recommandation          | Commande                |
| --------------------------------------- | ----------------------- | ----------------------- |
| Mettre à jour une branche locale        | `rebase`                | `git pull --rebase`     |
| Intégrer une feature dans develop       | `merge` (via PR ou CLI) | `git merge ma-feature`  |
| Mettre à jour une feature avec develop  | `rebase` (en local)     | `git rebase develop`    |
| Fusionner release dans main             | `merge`                 | `git merge release/x.y` |
| Fusionner main dans develop (après tag) | `merge`                 | `git merge main`        |



------

### ✅ Bonnes pratiques pour une petite équipe :

- Chacun travaille sur sa branche locale, met à jour via `rebase`, et crée une PR propre.
- Les merges se font via **pull request** vers `develop` ou `main`, pas directement dans le terminal (sauf cas de force majeure).
- Utiliser des **squash merges** pour aplatir les commits inutiles si besoin.