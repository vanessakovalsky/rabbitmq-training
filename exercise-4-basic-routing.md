# Exercice 4 - Routing de base

## Introduction

* Les PubSubs sont très pratique lorsque vous voulez distribuer à l'aveugle les message envoyée au même exchange à toutes les queues. Parfois vous aurez besoin d'un contrôle plus fin sur quel message est envoyé à quelle queue. Dans cette section, nous allons apprendre à router les messages à des queues spécifiques en utilisant le type d'exchange `direct` et les clés de routing.

## Objectif

* Router des messages à des queues spécifiques en utilisant une clé de routage

## Créer un exchange, deux queues et publier un message

* Créer un nouveau exchange `direct`:

```
rabbitmqadmin declare exchange name=morpheus type=direct
```

* Les exchanges `direct` permettent de faire correspondre des queues à des clé spécifiques (AKA `routing key`) et de router des messages à la queue pertinent en fonction de la clé fourni par le producteur
* Voyons le en action.

* Déclarer deux nouvelles queues:

```
rabbitmqadmin declare queue name=blue
rabbitmqadmin declare queue name=red
```

* Nous allons maintenant faire correspondre la queue `blue` pour ne recevoir que les messages associés à la clé de routage `steak`; la queue `red` recevra seulement les massages associés à la clé de routage `matrix`.

```
rabbitmqadmin declare binding source=morpheus destination=blue routing_key=steak
rabbitmqadmin declare binding source=morpheus destination=red routing_key=matrix
```

* Nous pouvons voir les nouveaux bindings avec les clé de routage dans la console Admin : 

![Bindings](/images/basic_routing/mgmt-1.png)

* Si votre binding d'exchange `morpheus` ressemble à celui de la capture, vous avez gagner le droit de publier un message :)

* A l'inverse de l'exchange `fanout` il est nécessaire de fournir une clé de routage pertinente - qui est associée avec un binding existant.  

```
rabbitmqadmin publish exchange=morpheus routing_key="steak" payload='I want to remain ignorant!'
rabbitmqadmin publish exchange=morpheus routing_key="matrix" payload='I want to see the light!'
rabbitmqadmin publish exchange=morpheus routing_key="neo" payload='going nowhere!'
```

* Nous pouvons observer deux choses :   

1. Les deux premiers messages sont publiés sans alerte. Le CLI Admin reboit le message suivant "Message published". Le dernier message, à l'inverse, retourne un message légèrement différent :"Message published but NOT routed".  
1. Aller sur l'écran [queues](http://localhost:15672/#/queues) montre que seulement deux messages sont maintenant dans les queues `blue` et `red` - un dans chaque queue.

* Qu'est t'il advenu du dernier message ? Comme la clé utilisée n'a jamais été déclaré sur aucun binding, l'exchange ne sait pas quoi faire et l'a pratiquement fait disparaître. C'est pour cette raison que nous avons un warning dans le CLI.

__Exercice:__ essayer de retrouver les messages dans les queus et de valider que le bon message a été dans la bonne queue (en fonction de sa clé de routage).

## Une clé; Plusieurs queues

* Il est possible de faire correspondre la même clé de routage à plusieurs queues. Essayons :

1. Créer une nouvelle queue appelé `blue-2`
1. Faire correspondre `blue-2` a l'exchange `morpheus` en utilisant la clé de routage `steak`.
1. Publier un autre message en spécifiant la clé de routage `steak`.

* Le message devrait être répliqué à la fois dans  `blue` et dans `blue-2`. Vérifier dans l'UI Admin!

## Une queue; Plusieurs clés

* Il est aussi possible de faire correspondre une queue en utilisant plusieurs clé de routage. Il suffit de lancer la commande `declare binding` plusieurs fois - chaque fois avec une clé différente:

```
rabbitmqadmin declare binding source=morpheus destination=blue routing_key=steak
rabbitmqadmin declare binding source=morpheus destination=blue routing_key=wine
```

* Un exemple plus pertinent peut être trouvé dans le [tutoriel officiel](https://www.rabbitmq.com/tutorials/tutorial-four-python.html). Voici une capture d'écran :

![Log Levels](/images/basic_routing/tutorial-1.png)

## Conclusion

* Vous avez maintenant les connaissances nécessaire pour construire une infrastructure de messagerie où vous avez un contrôle fin sur quel message va dans quel queue. 
* Utiliser l'exchange `direct` est particulièrement utile lorsque la classification des messages par destinations (comme les queues) est principalement statique, et que le nombre de clé de routage est faible. Malheureusement, ce n'est pas toujours le cas. Dans l'exercice suivant nous approndrons des manières plus avancées opur router les messages