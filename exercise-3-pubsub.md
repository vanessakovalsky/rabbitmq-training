# Exercice 3 - Pub / Sub

## Introduction

* Vous avez surement entendu parler du modèle de conception Pub/Sub design avant, mais si vous ne le connaisez pas, merci de lire [cet article](https://abdulapopoola.com/2013/03/12/design-patterns-pub-sub-explained/) avant de commencer.

* Les agens de messages comme RabbitMQ sont conçus pour implémenté le modèle Pub/Sub dans différents systèmes. Dans cette section nous allons implémenter un simple Pub/Sub au dessus de RabbitMQ en utilisant le type d'exchange `fanout`.
* Chaque exchange appartient à un type ou à un autre. Les types d'exchanges diffèrent dans la manière dont il route les messages aux queues.
* Dans l'exercice précédent nous avons utiliser l'exchange par défaut qui un exchange de type `direct`. Nous reviendrons aux exchanges de type  `direct`, pour l'instant, concentrons nous sur le type d'exchange `fanout`.

## Objectif

* Implémentant un Pub/Sub en utilisant un exchange `fanout`

## Créer un nouvel exchange

* Commençons par créer un nouvel exchange via le CLI : 

```
rabbitmqadmin declare exchange name=pubsub type=fanout
```

* Vérifions que notre echange est bien créé avec la commande :
```
rabbitmqadmin list exchanges name
```

* Notre nouvel exchange `pubsub` devrait être listé/ Vous pouvez aussi le voir dans l'[UI](http://localhost:15672/#/exchanges):

![Exchanges](/images/pubsub/mgmt-1.png)  

* Pour l'instant publier un message dans cet exchange ne fait rien, car il n'y a pas de queue _lié_(bound) à celui-ci. Afin d'obtenir les messages dans des queues, nous avons besoin de les faire correspondre avec les exchanges pertinent. Voyons comment le faire.

## Lier un exchange à une queue via un Binding

* Déclarer deux nouvelles queues : 

```
rabbitmqadmin declare queue name=sub_1
rabbitmqadmin declare queue name=sub_2
```

* Maintenant faisons les correspondre à l'exchange `pubsub` en déclarant un nouveau *Bindings*:

```
rabbitmqadmin declare binding source=pubsub destination=sub_1
rabbitmqadmin declare binding source=pubsub destination=sub_2
```

* Comme pour les queuex et les exchanges, vous pouvez aussi lister les bindings:

```
rabbitmqadmin list bindings
```

* Si tout est ok, vous devriez voir le nouveau binding dans la liste qui s'affiche. 
* Les bindings peuvent aussi être vue dans l'interface d'administration : 
    * Aller dans la page [exchanges](http://localhost:15672/#/exchanges) et sélectionner l'exchange `pubsub`. Puis déplier la section Binding (si elle n'est pas déjà dépliée):

![Bindings](/images/pubsub/mgmt-2.png)  


## Publier un message

* Pour la dernière partie, nous allons publier un message dans notre exchange `pubsub` et voir ce qu'il se passe:

```
rabbitmqadmin publish exchange=pubsub routing_key="" payload='this is spam'
```

* Revenir dans l'interface dans l'onglet [Admin UI > Queues](http://localhost:15672/#/queues). Noter que `sub_1` et `sub_2` ont chacune 1 message dans l'état _Ready_.

![Ready](/images/pubsub/mgmt-3.png)

* Vous pouvez maintenant consommer ces messages (ci-besoin voir l'exercice 1 pour savoir comment faire) et voir que ce sont les mêmes message!

* Ce qui s'est passé lorsque nous avons publié un message à l'exchange `pubsub` est qu'il a été distribué automatiquement à toutes les queues qui ont **souscrit** (ou sont liées) à celui-ci.

***Note***: Vous avez peut être remarqué que la `routing_key` passé à la commande de publication est vide. Cela fait sens puisque nous voulons que le message atteigne tous les soucripteur lorsque nous implémentons un PubSub. Pourquoi devons nous alors le passer la première fois? Car le [protocole AMQP](https://www.rabbitmq.com/resources/specs/amqp0-9-1.pdf) - qui est le protocole de communication utilisé par RabbitMQ - dit que nous devons le faire. En effet l'exchange `fanout` l'ignore seulement.
