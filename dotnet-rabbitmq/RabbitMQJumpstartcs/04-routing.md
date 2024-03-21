# 04 `Routing`

On veut envoyer un `Message` à différents `Services`.



## Utilisation d'un `Direct Exchange`

<img src="assets/direct-exchange-schema-exchange-message-binding-multiple.png" alt="direct-exchange-schema-exchange-message-binding-multiple" />

Le `Default Exchange` est un `Direct Exchange` sans nom.

Un `Direct Exchange` utilise des `Binding Keys` et des `Routing Keys` pour intelligemment router le `Message`.

Un `Binding` doit être créer entre une `Queue` et un `Exchange`. On doit définir une `Binding Key`.

La `Routing Key` doit correspondre à la `Binding Key` souhaitée.

Une `Queue` peut avoir de multiple `Binding Keys` et plusieurs `Queues` peuvent avoir la même `Binding Key`,  ce qui rend le système très souple.



## Utilisation d'un `Topic Exchange`

