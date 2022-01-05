# Exercice 8 - Plugins

## Objectifs
Cet exercice a pour objectif : 
* de savoir activer un plugin déjà présent
* de savoir trouver et installer un plugin tier

## Activation d'un plugin du coeur de RabbitMQ

* RabbitMQ fournit lors de son installation [différents plugins](https://www.rabbitmq.com/plugins.html#tier1-plugins) qui ne sont pas activés
* Nous allons activer le plugin sur le noeud rabbitmq1

### Activer les traces

* Pour que le plugin fonctionne, nous avons besoin que la fonctionnalité *[Firehose](https://www.rabbitmq.com/firehose.html)* qui permet de tracer les envois et accusés soit activée.
* Pour cela, une fois connecté sur le conteneur, lancer la commande : 
```
rabbitmqctl trace_on 
```
* Cela active le plugin sur le noeud par défaut et le vhost par défaut, on peut le modifier avec les options : 
    * -n pour le noeud
    * -p pour le vhost
* Vous devez mainteant activer les messages de traces en utilisant l'exchange prévu pour *amq.rabbitmq.trace*. Celui ci permet de tracer selon les routes suivantes :
    * *#* trace chaque message envoyé à tous les exchanges et livré dans toutes les queues
    * *publish.#* trace chaque message envoyé à n'importe quel exchange
    * *deliver.#* trace chaque message livré à n'importe quelle queue
    * *publish.X* trace chaque message envoyé à l'exchange X

### Activer le plugin
* Maintenant que les traces de Firehose soit visible dans l'interface graphique, nous activons le plugin *rabbitmq_tracing* avec la commande (depuis le conteneur de rabbitmq1):
```
rabbitmq-plugins enable rabbitmq_tracing
```
* Une fois activé vous retrouvé les traces dans l'interface graphique
![UI tracing](https://blog.rabbitmq.com/assets/images/2011/09/tracing.png)

::: tip Note
Cette fonctionnalité est pratique pour debugguer mais ne doit pas être utilisée en production car elle ralentit les performances, penser donc à ne pas l'activer en production.
:::

## Installation d'un plugin communautaire

* De nombreux plugins fournis par la communauté existe également et sous trouvable sur internet.
* Pour les installer, nous avons besoin du fichier .ez du plugin qui est le binaire à récupérer.
* En fonction de l'environnement dans lequel est exécuter votre serveur RabbitMQ, le chemin sera différent. 
[Liste des chemins en fonction de l'environnement](https://www.rabbitmq.com/installing-plugins.html)
* Nous allons installer un plugin qui permet de programmer la publication du message . Il s'agit de [rabbitmq-delayed-message-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)
* Pour l'installer se connecter sur le conteneur et exécuter les commandes suivantes : 
```
apt update
apt install curl -y
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.9.0/rabbitmq_delayed_message_exchange-3.9.0.ez 
mv rabbitmq_delayed_message_exchange-3.9.0.ez  $RABBITMQ_HOME/plugins/
chown rabbitmq:rabbitmq $RABBITMQ_HOME/plugins/rabbitmq_delayed_message_exchange-3.9.0.ez
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```
* Le plugin est activé il ne reste plus qu'à l'utilisé.