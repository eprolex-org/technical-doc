# AA Installation de `RabbitMQ`



## Sur `MacOS` avec `homebrew`

```bash
brew update

brew install rabbitmq
```

Si on doit mettre à jour

```bash
brew services stop rabbitmq

brew update

brew upgrade
```

Ensuite pour rendre disponible les commandes on ouvre le fichier `.zshcr` :

```bash
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH

# Path to your oh-my-zsh installation.
export ZSH="$HOME/.oh-my-zsh"

# Path for RabbitMQ
export PATH=$PATH:/usr/local/sbin

# ...
```

On y ajoute `export PATH=$PATH:/usr/local/sbin`

On lance ensuite le `server` : `rabbitmq-server`

<img src="assets/start-rabbit-mq-server.png" alt="start-rabbit-mq-server" style="zoom:50%;" />

Taper `stop` pour terminer.

On peut aussi utiliser `Homebrew` pour démarrer ou éteindre le service :

```bash
brew services start rabbitmq
```

```bash
brew services stop rabbitmq
```

Il est aussi recommandé d'activer les `flags` :

```bash
# highly recommended: enable all feature flags on the running node
rabbitmqctl enable_feature_flag all
Enabling all feature flags ...
```

J'ai dû faire :

```bash
/usr/local/sbin/rabbitmqctl enable_feature_flag all
```

> Grâce à `where` j'ai pu trouver la localisation de `rabbitmqctl` :
>
> ```bash
> ❯ where rabbitmqctl
> /usr/local/sbin/rabbitmqctl
> ```

Pour vérifier que le `service` tourne :

```bash
brew services list
```

<img src="assets/show-if-service-is-running.png" alt="show-if-service-is-running" />



Pour avoir les `infos` d'installation on tape :

```bash
brew info rabbitmq
```

<img src="assets/rabbit-mq-manager-url.png" alt="rabbit-mq-manager-url" />

On récupère ainsi l'`url` de l'intreface `admin` de `rabbitmq`.

User: `guest`

Password: `guest`

<img src="assets/rabbit-mq-intreface-admin-management.png" alt="rabbit-mq-intreface-admin-management" />



## Installation avec `Docker`

Configurer `user` et `password`:

```bash
$ docker run -it --rm --name my-rabbit -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=hukar -e RABBITMQ_DEFAULT_PASS=huk@r99_ rabbitmq:4.0-management
```

> Avec les paramètres par défaut :
>
> ```bash
> # latest RabbitMQ 4.0.x
> docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management
> ```

Si l'image n'est pas présente localement, `docker` la télécharge.

<img src="assets/docker-image-rabbit-mq-running.png" alt="docker-image-rabbit-mq-running" />

### ! Il faut spécifier les ports manuellement

<img src="assets/ports-settings-rabbitmq.png" alt="ports-settings-rabbitmq" style="zoom: 25%;" />

Sinon les ports du `Container` sont réassignés arbitrairement.



## Installation de `RabbitMQ` avec `Docker Compose`

Créer un fichier `docker-compose.yml`

```yml
version: '3.8'
services:
  rabbitmq:
  	container_name: "rabbitmq"
  	image: rabbitmq:3.8-management-alpine
  	environment:
  		- RABBITMQ_DEFAULT_USER=user
  		- RABBITMQ_DEFAULT_PASS=mypassword
  	ports:
  		# RabbitMQ instance
  		- '5672:5672'
  		# Web interface
  		- '15672:15672'
```

#### ! n'accepte pas les `tab` mais les `space` (2 par décalage).

 `management` pour avoir l'interface utilisateur.

On défini deux `ports`, un pour l'instance de `RabbitMQ` et un pour l'`interface web`.



### Utiliser `docker-compose.yml`

```bash
docker-compose up
```

Cette commande se charge de télécharger tout ce qui est nécessaire (normalement seulement la première fois).

On peut se connecter sur l'interface avec l'`URL` : http://localhost:15672/

<img src="assets/connect-to%20rabbitmq-interface.png" alt="connect-to rabbitmq-interface" />

Username : `user`

Password : `mypassword`

Une fois loggué:

<img src="assets/rabbitmq-user-interface-cool.png" alt="rabbitmq-user-interface-cool" />





















