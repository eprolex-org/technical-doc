
# 01 Les concepts de bases

Les `API HTTP` sont synchrone et la requête reste bloquée jusqu'à l'obtention de la réponse.
L'utilisation de `Message broker` permet de réellement fonctionner en asynchrone.

## Les choses à considérer
Comparer au système "simple" d'une `API HTTP` fonctionnant sur une base `requset-response`, implémenter une communication asynchrone avec un `message broker` ammène à réfléchri à de nouveaux points :
- La complexité augmente
- L'ordre des messages (avec plusieurs `Queues`)
- La duplication de message
- Créer un processus de message `Idempotent` (la même action donne le même état)
- Les garanties de livraison du message
- Lier à une technologie (par exemple `RabbitMQ`), utiliser `MassTransit` est une abstraction répondant à ce problème.

## `async` messaging

- Les composants peuvent opérer indépendamment
- On a une grande flexibilité
- On peut gérer de gros volume de messages
- Le système a une meilleur tolérance aux problèmes (un `Consumer` pouvant être relancé si besoin sans que le message dans la `Queue` ne soit perdu)

Contrairement aux `API HTTP`, ce système n'a pas besoin que deux partie soient disponible en même temps.
Une analogie avec le monde réel serait un serveur de restaurant, il ne doit pas attendre que l'on vous serve les entrées pour prendre la commande des plats et des desserts.
Avec une architecture en `async messaging`, il est possible de lancer plusieurs demandes sans devoir attendre que l'une soit finie avant l'autre.

### Changement des rôles
Client => Sender/Producer/Publisher
Server => Receiver/Consumer

### Avantages
- scalability (horizontale) avec une `API HTTP` cela serait plus compliqué et nécessiterait un `Load Balancer`.
- Des composants peu couplés.
- La tolérance aux erreurs (Fault Tolerance) et la resistance au pannes (resilience). Les messages sont enregistrés et ré-envoyé dès qu'un service est de nouveau disponible. Les messages ne sont jamais perdus.
- Intégration : permet la communication entre des technologies hétéroclites.
- La monté en charge ne dépend pas du `Consumer`.
- Découplage temporel : les composants ne sont pas obligé de travailler en même temps.

## le `message`
Il est composé du `header` et du `payload`.

### `Header`

Rempli par le système de message ou manuellement.

### `Payload`

Ce sont les données métiers.

Dans `Masstransit`, un `message` peut être une `classe`, un `record` ou une `interface`.

Le type ne doit avoir que des `propriétés` et pas de comportement, il doit être un type `référence`.

Le `Content-Type` défini dans le `header` par `Masstransit` est `application/vnd.masstransit`.

Une bonne pratique est de mettre les types de `message` dans une librairie séparée pour pouvoir facilement les partager.

Ces types sont appelés `contract`.



## Les modes/guaranties de livraison

- Le message délivré a un identifiant unique.
- Un état de livraison est maintenu pour s'assurer que le message est livré un nombre de fois spécifié.



### 3 modes de livraison

- Au plus une fois => `pas d'accuser de reception`, on ne vérifie pas que le message a bien été reçu. Peu fiable mais très performant.
- Au moins une fois => `avec accusé de reception`, le `broker` s'assure que le message a été délivré au `consumer` au moins une fois. Le `broker` attend après un `accusé` de la part du `Consumer`. Le `broker` renvoie le message après un certain temps. Le `message` peut donc être traité pluseiurs fois si le `Consumer` crash juste avant d'envoyer l'`accusé`.
- Exactement une fois => Certain `broker` possède un système de `transaction` garantissant le traitement de une fois exactement. Cela peut poser des problème de débit (`throughput`). Cela peut aussi être réalisé grâce au pattern `inbox/outbox` ou à une implémentation `idempotente`.



## Topologies

Cela se réfère à la manière dont la configuration et la structure du routage de `message` est mis en place avec le `message broker`.

La `topologie` utilise les types pour configurer l'infrastructure du `broker`.

Elle est utilisé aussi pour accéder au capacités du `broker` (`exchange`, `routing key`,  ...).

Dans `Masstransit` chaque `Queue` correspond à un `Consumer` ou un groupe de `Consumer` spécifique.

`Masstransit` simplifie et abstrait la `topologie` des `brokers` tout en permettant une configuration fine si besoin.

On peut choisir une `topologie` : `automatic` où `Masstransit` s'occupe de cela à notre place, ou bien une `topologie` : `custom` où l'on configure sois-même les choix topologique.



## `Endpoints`

Les `endpoints` dans `Masstransit` représente des adresses ou destination depuis lesquelles les messages sont envoyés ou reçus.

Par défaut `Masstransit` ne nécéssitte aucune configuration des `endpoints`, il suffit d'appeler la méthode `Configure Endpoints`.

La configuration de `Masstransit` est basée sur les conventions.

Utiliser la configuration automatique en appelant `Configure Endpoints` est fortement recommandé.

1. `Receive endpoint` c'est la destination pour consommer les messages. C'est en général une `Queue` où le `Consummer` écoute les `messages` à traiter.
2. `Publish endpoint` Il publie des `événements` pour tous les `subscribers` intéressés. Il utilise l'interface `IPublish`.
3. `Send endpoint` une destination pour envoyer les `messages`. Il représente l'`adresse` où les `messages` sont envoyés sans avoir à ce soucier des `Consumers`.

Les `endpoint` peuvent être configurer pour définir :

- Les détails de `transport`
- La `concurrence` et `Prefetch Limits`
- La politique de `retry`

















