# Exercice 9 - Haute disponibilité

## Objectifs
* Cet exercice a pour objectif de comprendre les différentes possibilité pour rendre les queues hautement disponible
* Cette exercice a pour objectif la mise en place d'une queue en miroir sur nos différents noeuds de cluster

## Quelles possibilités pour rendre nos queues hautement disponibles

* La mise en place d'un cluster de noeuds ne suffit pas à rendre notre cluster hautement disponible. En effet les queues ne sont pas répliquées par défaut ce qui entraine la perte de données dans le cas du crash complet d'un noeud. 
* Il existe 3 possibilités pour les queues pour les rendre hautement disponibles : 
    * les queues miroirs : il s'agit d'une réplication complète d'une queue et de ses messages sur chacun des noeuds du cluster. Son inconvénient majeur est qu'il faut attendre que les messages soient complètement dupliqués pour pouvoir utilisé la queue et notamment récupérer les messages de celle-ci. Ce type de réplication sera supprimé dans une version future de RabbitMQ il n'est donc pas conseillé de l'utiliser.
    * les queues quorum sont également des queues qui permettent de répliquer les queue FIFO en s'appuyant sur l'[algorythme du consensus de Raft](https://raft.github.io/). Celles-ci s'appuie sur un nouveau design qui entraine également des limitations mais elles ont été pensé pour être plus facile à utiliser et plus sécurisée. [La liste des fonctionnalités des queues quorum](https://www.rabbitmq.com/quorum-queues.html#feature-matrix).
    * les streams sont une nouvelle (depuis RabbitMQ 3.9) structure de données persistante. Ceux-ci sont utiles dans 4 cas : 
        * fan-outs large: dans le cas où vous avez des messages à distribuer à de nombreux client. 
        * Replay / time-travelling : pas de destructions de messages ce qui permet aux consommateurs de les retrouver à partir d'un point spécifique
        * Besoin de performance : les stream ont été conçu avec comme objectif principal la performance, ce qui permet de traiter des volumes de données important avec une excellente performance de traitement
        * Large logs : Volumes de données important avec des millions de messages dans une queue.  
* Ces trois types propose de la réplication et permettent ainsi de s'assurer qu'en plus du serveur qui est répliquer avec les noeuds du cluster, nos données et leur état sont bien sécuriés et répliqués.

## Création et usage d'une queue de type quorum

* Nous allons créer une queue de type quorum. Voici la commande à utiliser (cela peut également être fait dans l'UI) :
```
rabbitmqadmin declare queue name=quorumqueue durable=true arguments='{"x-queue-type": "quorum"}'
```
* Vérifier sur chaque noeud que la réplication de la queue s'est bien créé 
* Créer un exchange (du type de votre choix) et un binding pour pouvoir envoyer des messages à votre queue
* Publier un message sur votre queue *quorumqueue*
* Le message devrait alors se répliquer sur les autres queues, est ce le cas sur l'ensemble des répliques de queue ?
* Que se passe t'il si vous consommez le message (sans le remettre dans la queue) ? 

-> A vous maintenant de réfléchir à la mise en place de queue hautement disponible sur votre cluster. Quelles sont les queues qui ont besoin d'être répliquées et sécurisées ? Quel type de queue utiliser pour cela ?