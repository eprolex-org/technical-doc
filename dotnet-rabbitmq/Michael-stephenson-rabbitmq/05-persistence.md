# 05 La `persistence`

## La persistence pour une `queue`

### `Durable`

- Le `message` est sauvé sur le disque
- Le `message` reste en vie après un redémarage serveur.
- Cela à un coût en performance



### `Non-Durable`

- le `message`. est gardé seulement en mémoire
- le `message` disparait après un redémarage serveur
- Les performances sont meilleures

Le `design` de l'application doit choisir entre performance et sécurité des `messages`.