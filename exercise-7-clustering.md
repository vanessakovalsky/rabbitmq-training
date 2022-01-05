# Exercice 7 - Mise en place d'un cluster RabbitMQ

## Objectifs 
Cet exercice a pour objectifs : 
* De créer les noeuds du cluster RabbitMQ
* De configurer RabbitMQ pour qu'il utilise ces noeuds.

## Mise en place des noeuds du cluster

* Afin de mettre en place notre cluster, nous allons utiliser 3 noeuds. 
* Pour cela on commence par arrêter notre conteneur :
```
docker-compose stop
```
* Puis on remplace le contenu du fichier *docker-compose.yaml* par le contenu suivant : 

```yaml
version: '3'

services:

  rabbitmq1:
    image: rabbitmq:3.9-management
    hostname: rabbitmq1
    ports:
      # The standard AMQP protocol port
      - '5672:5672'
      # HTTP management UI
      - '15672:15672'
    environment:
      - RABBITMQ_ERLANG_COOKIE=${RABBITMQ_ERLANG_COOKIE}
      - RABBITMQ_DEFAULT_USER=${RABBITMQ_DEFAULT_USER}
      - RABBITMQ_DEFAULT_PASS=${RABBITMQ_DEFAULT_PASS}
      - RABBITMQ_DEFAULT_VHOST=${RABBITMQ_DEFAULT_VHOST}
    networks:
      - network
  rabbitmq2:
    image: rabbitmq:3.9-management
    hostname: rabbitmq2
    depends_on:
      - rabbitmq1
    environment:
      - RABBITMQ_ERLANG_COOKIE=${RABBITMQ_ERLANG_COOKIE}
    networks:
      - network
  rabbitmq3:
    image: rabbitmq:3.9-management
    hostname: rabbitmq3
    depends_on:
      - rabbitmq1
    environment:
      - RABBITMQ_ERLANG_COOKIE=${RABBITMQ_ERLANG_COOKIE}
    networks:
      - network
networks:
  # Declare our private network.  We must declare one for the magic
  # Docker DNS to work, but otherwise its default settings are fine.
  network: {}
```
* Afin de ne pas répéter les variables sur les différents services, nous avons défini un fichier .env qui contient les variables que docker-compose utiliser. Voici son contenu : (le fichier doit être créé)
```
RABBITMQ_ERLANG_COOKIE=12345
RABBITMQ_DEFAULT_USER=guest
RABBITMQ_DEFAULT_PASS=guest
RABBITMQ_DEFAULT_VHOST=/
```
* Si vous aller sur l'interface d'administration dans les nodes, vous ne verrez que le node rabbit@rabbitmq1, c'est normal, nous avons besoin de dire à RabbitMQ de rattacher les deux autres serveurs au cluster.

## Rattacher les deux autres serveurs au cluster
* Se connecter sur le conteneur du service rabbitmq2 (adapté le nom du conteneur à votre environnement, utilisez la commande *docker ps* pour obtenir la liste des conteneurs et leur nom) :
```
docker exec -it rabbitmq-training_rabbitmq2 bash
```
* Puis nous arrêtons l'application et la rattachons au cluster avec les commandes suivantes :
```
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@rabbitmq1
rabbitmqctl start_app
```
* Allez vérifier sur l'interface d'administration ou avec le CLI que votre noeud apparait bien dans la liste des noeuds.
* Faire de même pour le service rabbitmq3 

-> Félicitations, votre cluster est prêt à être utilisé et vous pouvez dès à présent créer des queues, exchanges et bindings sur n'importe lequel de vos noeuds :)

**Note** : 3 nodes ou un nombre impair pour éviter le [split-brain](https://fr.wikipedia.org/wiki/Split-brain_(informatique) )