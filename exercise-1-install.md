# Exercice 1 - Installation et interface de Rabbit MQ

## Objectifs 
Cet exercice a pour objectifs : 
* de vous faire installer sur votre poste un RabbitMQ pour pouvoir le manipuler
* de découvrir l'interface graphique de RabbitMQ, la ligne de commande et les différents concepts de cet outil

## Pré-requis

* Docker doit être installé sur votre machine
* Docker compose doit également être installé

## Installation

* Puisque nous avons choisi d'utiliser RabbitMQ via une image  Docker, vous pouvez le lancer via  la commande suivante : 
```
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.9-management
```
* Une fois l'image téléchargée et le conteneur lancé, l'interface graphique web de l'application sera accessible sur http://[IP-ou-HOSTNAME-de-votre-DOCKER]:15672 

* Afin de simplifier la suite (notamment le développement d'une application java), il est nécessaire de créer un fichier docker-compose.yaml avec le contenu suivant : 
```
    version: "3.6"
    # https://docs.docker.com/compose/compose-file/
    services:
      rabbitmq:
        image: 'rabbitmq:3.9-management'
        ports:
          # The standard AMQP protocol port
          - '5672:5672'
          # HTTP management UI
          - '15672:15672'
        environment:
          # The location of the RabbitMQ server.  "amqp" is the protocol;
          # "rabbitmq" is the hostname.  Note that there is not a guarantee
          # that the server will start first!  Telling the pika client library
          # to try multiple times gets around this ordering issue.
          AMQP_URL: 'amqp://rabbitmq?connection_attempts=5&retry_delay=5'
          RABBITMQ_DEFAULT_USER: "guest"
          RABBITMQ_DEFAULT_PASS: "guest"
        networks:
          - network
    networks:
      # Declare our private network.  We must declare one for the magic
      # Docker DNS to work, but otherwise its default settings are fine.
      network: {}
```
* Puis nous lançons l'application avec la commande : 
```
docker-compose up -d 
```

* Ouvrir alors de nouveau la page, et vous obtenez une page ressemblant à celle-ci

!(Login RabbitMQ)[images/rabbitmq_login-min-1024x561.png]

* Les informations de connexion se trouvent dans les variables d'environnement définies dans le docker-compose.yaml
* Une fois connecté vous obtenez l'écran suivant :

!(Accueil RabbitMQ)[images/rabbitmq_dashboard-min-1024x563.png]

* Il est également possible d'installer RabbitMQ de nombreuses autres manières directement sur votre poste ou chez des fournisseurs, voir la documentation officielle ici : https://www.rabbitmq.com/download.html 
