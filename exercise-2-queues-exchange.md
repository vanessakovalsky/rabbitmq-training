# Exercice 2 - Les bases : Queues et Exchanges

## Objectif

* Cet exercice a pour objectif de publier notre premier message avec RabbitMQ et de le consommer via le CLI


## Créer une queue et publier un message

* Créer une nouvelle queue appelée `hello`:

```sh
rabbitmqadmin declare queue name=hello
```
* Les queues sont justes des tampons qui sont remplies de messages. Les messages sont soient stockés dans une queues soit pas du tout.
* Envoyer notre premier message : 

```
rabbitmqadmin publish exchange=amq.default routing_key=hello payload="hello, world"
```
* Les messages ne peuvent pas être envoyées directement aux queues. Ils arrivent dans les queus via les *exchanges*. Dans cet exemple, nous avons publiée un message sur le  **default exchange**. 
* L'exchange par défaut est implicitement lié à chaque queue, avec une clé de routing équivalente au nom de la queue. 

* Voyons ce que cela donne sur la console de gestion : 

* Aller sur [http://localhost:15672/#/queues](http://localhost:15672/#/queues). Vous devriez voir quelque chose comme ça :  
![Queues](/images/basics/mgmt-1.png)  

* Si le compteur *ready* affiche 0, il y a un soucis quelque part.


## Consommer notre message

### Avec la console de gestion 

* Commençons par inspecter le message que nous avons envoyé via la console de gestion : 
* Aller à http://localhost:15672/#/queues/%2F/hello ou cliquer seulement sur le nom de la queue, si vous n'avez pas quitter l'écran précédent.
* Cliquer sur l'onglet **Get Messages**   
![Get Messages Tab](/images/basics/mgmt-2.png)  

* Puis cliquer sur le bouton **Get Message(s)**
![Get Messages Button](/images/basics/mgmt-3.png)  
* Voici notre message qui s'affiche.

* Noter que l'option **Requeue** est définie à **Yes**. Cela signifie que les message reviendra dans la queue après que nous l'ayons reçu. En production les messages ne retourne normalement pas dans la queue, mais pour debugguer, cela est pratique.

### Via le CLI Admin
* Utilisons maintenant le CLI Admin pour lire le message :
```
rabbitmqadmin get queue=hello ackmode=ack_requeue_false
```
* Cette fois nous avons choisi de ne pas remettre dans la queue le message.

* Revenez dans la console de gestion vous devriez voir que votre queue `hello` est vide. Vous pouvez aussi utiliser le CLI:

```
rabbitmqadmin list queues name messages_ready
```
* Les deux dernier paramètre sont le nom des colonnes que vous voulez que le CLI affiche lorsqu'il liste les queues. Dans ce cas, nous voulons seulementle nom de la queue et le nombre de message dans l'état *ready*.

* Regarder les options de la commande `list queues` et essayez les. Cela pourrait vous être utile plus tard!
