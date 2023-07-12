# 01 Redux

`Redux` est un `design pattern` pour gérer les données du point de vue du `Front End`.



## Composant de `Redux`

- `State` c'est le magasin de données principale (`Data store`), il est `Immuable`.
- `Actions` C'est un message qui déclenche une modification du `State`.
- `Reducers` ce sont de `Pure Fonction` qui reçoivent les `Actions` pour modifier le `State` (en fait une nouvelle version du `State`).
  Une `Pure Fonction` a toujours le même résultat de sortie avec les mêmes paramètre en entrée.

Il existe aussi le concept des `Effects`.



## Deux concepts clés pour `Redux`

- `State` et `Actions` sont `Immuable`
- Les `Reducers` sont des `Pure Functions`

