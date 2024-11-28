
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


### `Payload`


















