# 09 `Workers` concurrents

Les `Consumers` doivent pouvoir être *scalé* de manière indépendante de l'`API` par exemple. Ils doivent donc avoir leur propre application, le plus souvent un `Worker Service`.

On veut pouvoir les faire travailler de manière concurrente.



## Configuration dans le `Worker Service`

