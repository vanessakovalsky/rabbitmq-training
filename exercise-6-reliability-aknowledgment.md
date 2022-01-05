# Exercice 6 - Fiabilisation des queues et accusé de reception / traitement

## Objectifs 
Cet exercice a pour objectifs : 
* de savoir gérer la persistence des queues
* de gérer les accusés de réception / de traitement 

## Gérer la persistance des queues

### Présentation 
* Une queue est un structure de données séquentielles. Elle sert à stocker les messages envoyées par le(s) producteur(s) avant qu'ils ne soient traité par un ou des consommateurs.
* Elles sont FIFO (Firt in, first out), mais ne garantissent pas l'ordre des messages. Cependant si l'option redelivered est mise à *false*, le consommateurs peut considéré que les messages sont livrés dans le même ordre que celui dans lequel ils ont été envoyé à la queue. Il est possible de définir une priorité pour chaque consommateur, à priorité égale c'est l'algorythme round-robin qui est utilisé pour déterminé l'ordre (https://en.wikipedia.org/wiki/Round-robin_scheduling)
* Chaque queue peut être persistante (durable) ou éphémère (transient). Il est indispensable d'utiliser des queues durables dans un environnement où l'on ne peut pas perdre de données (tout dépend donc des cas d'utilisation et du contenu des messages stockées dans les queues)
* Il est possible de créer des queues temporaires qui sont supprimées selon 3 critère : 
    * queues exclusives qui peut seulement être consommé, purgé et supprimé, depuis la connection qui a créé cette queue
    * TTLs : on peut définir une durée de vie à une queue, une fois le délais dépassé, la queue sera supprimé par RabbitMQ
    * Auto-delete queue : queue qui se supprime lorsqu'il n'y a plus aucun consommateur abonné à celle-ci.

### Différence entre une queue durable et une queue non durable

* Créer deux queues : l'une sera durable et l'autre transient
* Envoyer un message à chacune de ces queues (via le exchange par défaut)
* Vérifier que vos messages sont là en les remettant dans la queue
* Rédémarrer votre conteneur 
```
docker-compose restart
```
* Vérifier maintenant si vos deux queues sont toujours présentes ?

### Ordre des messages 
* Publier un message dans votre queue durable créé à l'étape précédente
* Sans visualiser le premier, publier un second message avec un texte différent
* Consulter maintenant le premier message, en replaçant le message lu dans la queue.
* Quel est l'ordre des messages ?

## Gérer les accusés de réception / de traitement 

### Présentation 
* Les accusés de réception . trzitement permettent à RabbitMQ de savoir si un message a bien été reçu par le consomateur et s'il a bien été traité ou non.
* Il existe trois types d'accusés possible : 
    * *ack* signifie que le message a été reçu et traité
    * *nack* signifie que le message a été reçu mais que son traitement n'a pas fonctionné. 
    * *reject* signifie que le message a été reçu, mais qu'il n'a pas pu être traité.
* Dans le cas d'une réponse négative (nack, ou reject) il est possible en passant le paramettre *requeue* à $true* de demander à RabbitMQ de remettre le message dans la queue.
* Afin d'avoir une garantie, il est fortement recommandé d'utiliser les confirmations pour savoir si un message a ou non été délivré, cela permet également au producteur du message d'être sûr que son message a été reçu et traité. 

### Utiliser les différents accusés

* Créer trois message dans la queue créé lors de l'étape précédente
* Lisez les 3 messages et utilisez pour chacun un accusé différents.
* Les messages sont t'ils remis dans la queue ?
* Quelle est l'option à activer pour que les messages soient remis dans la queue ?

