# 03 Guide de l'`api`

## `IConnection` et `IChannel`

`connection` et `channel` sont pensés pour avoir une longue durée de vie.

Un `channel` peut avoir une durée de vie plus courte que la `connection`, certain protocoles d'erreur vont fermer un `channel` automatiquement. Si l'application pet se remettre de l'erreur, un nouveau `channel` sera rouvert et ré-essayera l'opération.

Les `exceptions` au niveau d'un `channle` provoquent la fermeture du `channel`.