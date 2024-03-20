# 03 `Pub/Sub`

Délivrer le même `Message` à plusieurs `Consumer`.

Dans le cas où une personne créé un nouvel acompte, il est possible que plusieurs `Serrvices` soit concerné par cet événement.



## `Fan Out Exchange`

C'est en jouant sur l'`Exchange` que l'on va réaliser ce `pattern Pub/Sub`.

`RabbitMQ` ne stocke pas plusieurs copies du `Message`, il le stocke une seule fois et chaque `Queue` reçoit une référence du `Message`.

Le `Producer` ne s'inquiète oas de savoir combien de `Services` vont recevoir son `Message`, `0` ou `9` peut importe, il envoie son `Message` au `Fanout Exchange` et c'est tout.

C'est le `Fanout Exchange` qui s'occupe de redistribuer le `Message` dans les `Queue` abonnée. Les `Queues` sont `Bindées` à l'`Exchange` qui les intéresse.