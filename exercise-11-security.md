# Exercice 11 - Sécurité

## Objectifs 
Cet exercice a pour objectif : 
* De gérer les utilisateurs
* De définir les policies 

## Gérer les utilisateurs et les permissions

* RabbitMQ propose nativement de la gestion d'utilisateurs que l'on peut limiter par virtualHost.
* Pour avoir des utilisateurs avec des permissions plus spécifique, on peut activer le plugin rabbitmq_auth_backend_http . Pour cela, se connecter sur le serveur rabbitmq (le conteneur) et exécuter la commande suivante pour activer le plugin :
```
rabbitmq-plugins enable rabbitmq_auth_backend_http 
```
* Il est également possible dans la configuration d'[activer d'autres système d'authentification]
(https://www.rabbitmq.com/access-control.html#combined-backends)

* Nous allons maintenant créer un utilisateur (depuis l'interface ou avec la commande suivante) :
```
echo '2a55f70a841f18b97c3a7db939b7adc9e34a0f1b' | rabbitmqctl add_user 'username'
```
* Pour les permissions, il est possible de définir pour un vhost donné des permissions pour : 
    * configurer les echanges et les queues
    * écrire dans les exchanges et les queues
    * lire dans les exchanges et les queues
* La définition de permissions se fait sur la base d'expression régulière pour chacun de ces éléments. 
* On peut définir des actions spécifiques qui sont autorisés via la [liste d'opération AMQP](https://www.rabbitmq.com/access-control.html#authorisation)
 ou bien mettre une étoile pour donner accès à toutes les opérations d'une des catégories de permissions. 
* Il est recommandé de définir une stratégie de nommage avec des prefixes ou des suffixes pour les exchanges et les queues pour faciliter la gestion des permissions et des policies
* Ajouter les permissions de configurer à notre utilisateur seulement lorsque l'exchange ou la queue commance avec son nom : 
```
rabbitmqctl set_permissions -p "custom-vhost" "toto" "toto\..*" "toto\..*" ".*"
```
* Le plugin *rabbitmq_auth_backend_ip_range* permet de faire de l'autorisation basée sur des adresses IP des clients.    

## Définir une police

* Les policies permettent d'appliquer certaines options aux queues et/ou aux exchange sans avoir besoin de les déclarer à chaque nouvelle création.
* L'avantage des policies est que l'on va aussi pouvoir modifier leur valeur, ce qui n'est pas possible sur les options et arguments d'une queue ou d'un exchange une fois qu'il a été créé.
* Pour définir une police, il est possible de le faire de différentes façons :
    * via le cli avec la commande *rabbotmqctl set_policy*
    * via l'interface graphique
    * via un appel à l'API rest
* Un exemple de commande : 
```
rabbitmqctl set_policy HA '^(?!amq\\.).*' '{"ha-mode": "all"}'
```
* Les arguments utilisés sont les suivants : 
    * *HA* : nom de la police
    * *'^(?!amq\\.).*'* : le pattern sur lequel on applique cette police
    * *'{"ha-mode": "all"}'* : la règle a appliquer (ici on active le miroring classique des queues)
* Le même exemple via l'API REST :
```
PUT /api/policies/%2f/HA
    {"pattern": "^(?!amq\\.).",
     "definition": {"ha-mode": "all"}
    }
```
* Vous pouvez donc définir des options qui s'appliqueront à tout ou partie de vos queues par ce biais.

**A noter** : Aujourd'hui il n'est pas possible de définir le x-queue-type via les policies, donc de forcer des queues à être de type quorum ou stream, cela doit être fait au moment de la création de la queue.

## Bonus - Paramétrer un certificat de sécurité

Pour paramètrer le certificat de sécurité on peut suivre cette documentation : https://blog.zwindler.fr/2019/04/16/suivez-le-lapin-orange-intro-et-bonnes-pratiques-dinfra-rabbitmq/#comment-activer-le-chiffrement-tls-sur-mes-interfaces-rabbitmq-  