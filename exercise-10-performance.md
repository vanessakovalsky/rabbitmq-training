# Exercice 10 - Performance

## Objectifs

Cet exercice a pour objectifs :
* De découvrir un outil pour analyser les performances de notre cluster
* De connaitre les moyens d'optmiser les performances de notre cluster

## Installation et utilisation de l'outil d'analyse de performance de RabbitMQ

* L'outil [RabbitMQ PerfTest](https://rabbitmq.github.io/rabbitmq-perf-test/stable/htmlsingle/) permet de mesurer les performances d'un serveur RabbitMQ
* Il est nécessaire d'installer l'outil (sur une machine Java) et de lui permettre de se connecter à notre serveur RabbitMQ.
* Nous allons seulement le rajouter en tant que service dans notre docker-compose.yaml :
```yaml
[...]
  perftest:
    # https://hub.docker.com/r/pivotalrabbitmq/perf-test/tags
    image: pivotalrabbitmq/perf-test:latest
    networks:
      - network
    environment:
      URI: "amqp://guest:guest@rabbitmq1:5672/%2f"
[...]
```
* Maintenant nous lançons notre nouveau conteneur pour mettre en route notre outil de test de performance : 
```
docker-compose up -d
```

* Notre outil est prêt à être utilisé.
* Connecter vous sur le conteneur
```
docker exec -it rabbitmq-training_perftest_1 bash
```
* Puis lancer une commande pour executer un test, celle-ci simule un producteur unique sans confirmations, deux consommateurs (chacun recevant chaque message) qui utilisent l'acquittement automatique et une queue unique nommé *throughput-test-1-x1-y2*. Le producteur publie aussi vite que possible sans limite de message. 
 ```
bin/runjava com.rabbitmq.perf.PerfTest -x 1 -y 2 -u "throughput-test-1" -a --id "test 1"
```
* Pendant que le test tourne vous pouvez suivre la consommation dans l'UI et/ou voir les résultats dans le terminal.
* Plus de tests et d'options sont disponibles dans la [documentation](https://rabbitmq.github.io/rabbitmq-perf-test/stable/htmlsingle/)

## Optimiser la configuration
* Nous allons maintenant optimiser la configuration pour améliorer les performances. 
* Pour obtenir le chemin des fichiers de configs, après s'être connecter au conteneur de notre serveur RabbitMQ nous utilisons la commande *rabbitmq-diagnostics status*
* Celle-ci nous fournit différentes informations et parmi ces informations la localisation des fichiers de configuration, par exemple : 
```
Config files

 * /etc/rabbitmq/advanced.config
 * /etc/rabbitmq/rabbitmq.conf
```
* Concernant les fichiers de configuration il existe différents fichiers et leur format est différent :
    * rabbitmq.conf : formation systctl ou ini-like
    * advanced.config : formation classic (earlang terms)
    * rabbitmq-env.conf : paires de variable d'environnement

* Il est recommandé de déclaré la configuration dans le nouveau format mais toutes les configurations ne sont pas possible dans ce fichier et ce format, d'où le fait que l'ancien fichier et format soit encore présent. [Une liste des variables communes de ce fichier est disponible](https://www.rabbitmq.com/configure.html#config-items) 
* Pour que la configuration soit prise en compte il est nécessaire de redémarrer le serveur RabbitMQ, soit avec rabbitmqctl soit en redemarrant le conteneur.

* Afin d'optimiser les performances, en dehors du fait de rajouter des ressources sur vos machines, il est recommandé de suivre les [recommandations pour la production donnée par rabbitMQ](https://www.rabbitmq.com/production-checklist.html#resource-limits-ram)
* Appliquer les valeurs conseillées et relancer un test de perf pour voir la différence.

* Par la suite, il faudra monitorer l'infrastructure et le cluster RabbitMQ pour adapter au mieux ces valeurs.

* Pour optimiser au mieux vos performances, il est conseillé de chercher à savoir d'où viennent les lenteurs : 
    * I/O sur le disque
    * RAM 
    * Réseau 
* En fonction vous pourrez adapter les bons paramètres.
