# 04 `Queues`



## Caractéristiques

- `Name` : Le nom de la `Queue`
- `Durable` : Persister la `Queue` sur le `Disque`
- `Exclusive` : Supprimer la `Queue` lorsqu'on n'en a plus besoin
- `Auto Delete` : la `Queue` est supprimée lorsque le `Consumer` se désinscrit (`Unsubscribe`)



## `Bindings`

<img src="assets/binding-queues-schema.png" alt="binding-queues-schema" />

Un `Exchange` va utiliser la `Rounting Key` pour contacter la bonne `Queue`.

Elle possède des `Queues` qui lui sont liées.



## `Consumer`

Le `Consumer` est l'application consommant la `Queue`.

Il peut y avoir plusieurs `Consumer` pour une même `Queue`.

<img src="assets/publisher-consumer-schema-blue-green.png" alt="publisher-consumer-schema-blue-green" />



## Message `Acknowledgements`

Un `Message` est retiré de la `Queue` lorsqu'il est envoyé au `Consumer` ou lorsque le `Consumer` envoie un `Accusé de Reception` (`Acknowledgement`).

<img src="assets/acknowledgment-sent-to-queue-schema.png" alt="acknowledgment-sent-to-queue-schema" />



## `Rejecting` Messages

Le `Consumer` peut rejeter le `Message`.

Le `Message` est soit retiré (`Discard`) soit remis dans la `Queue` (`Re-Queue`).

Si le `Message` est remis dans la `Queue`, il faut faire attention aux boucles infinies.

<img src="assets/rejecting-the-message-schema.png" alt="rejecting-the-message-schema" />





















