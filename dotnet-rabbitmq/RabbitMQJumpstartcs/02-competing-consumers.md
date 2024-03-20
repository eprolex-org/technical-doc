# 02 `Consumers` concurrents

Ce pattern est utilisé pour des tâche assez longue: `image processing`, `machine learning`.

Au lieu d'attendre la fin de la tâche, on envoie celle-ci vers des `Workers` via la `Queue` qui la réalise pour nous.

Pour le `web` cela permet de ne pas bloquer la petite fenêtre d'une communication `HTTP`.

Cela améliore aussi la `scalability` et la `reliability` (fiabilité) du système.



## Le problème

Si un `Producer` envoie une tâche toute les `5s` et que le `Consumer` met `10s` à la réaliser, cela créer une accumulation dans la `Queue` (`backlog`), le `Broker` peut à terme être `Out Of Memory`.

<img src="assets/the-problem-explian-in-schema.png" alt="the-problem-explian-in-schema" />

On peut ajouter un nouveau `Producer`:

<img src="assets/pattern-competing-consumers-round-robin-messages-delivery.png" alt="pattern-competing-consumers-round-robin-messages-delivery" />

`rabbitMQ` par défaut gère le système en `Round-Robin`, chacun son tour.

Ce n'est pas l'approche la plus efficace si chaque `Producer` traite le `Message` dans un temps différent le `Producer A` aura `2` message en attente (dans la `Queue`) alors que le `Producer B` en aura `0` :

<img src="assets/round-robin-pattern-limit-of-efficiens.png" alt="round-robin-pattern-limit-of-efficiens" />

On doit régler la valeur de `prefetch` à `1` pour que `RabbitMQ` n'envoie pas plus. de `1` message par `Worker`.

Ce pattern permet de faire tourner plus de `Worker` si nécessaire (`scalability`) et permet que si l'un crash, il reste toujours au moins un autre `Worker` pour acomplir la tâche (`reliability`).

<img src="assets/one-worker-crashed-and-two-syill-alive.png" alt="one-worker-crashed-and-two-syill-alive" />







