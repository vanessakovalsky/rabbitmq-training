# Exercice 5 - Routage multidimensionnel avec les Topics

## Introduction

* Dans l'exercice précédent, nous avons appris à utiliser les exchanges `direct` et les clés de routage pour router les messages vers des queues spécifiques. Aussi puissant que cela est, cette méthode a une limitation majeur - il permet du routage basé seulement sur **une** dimension.

* Pour comprendre ce que cela signifie, prenons un exemple hypothètique : supposons que nous avons un système qui suive le table de temps de tous les vols - domestiques et internationaux - aux US. Le système reçoit le programme d'un vol et l'envoi à différents consommateur en utilisant RabbitMQ comme agent de message. Au début vous voulez envoyer tous les vols domestiques sur une seule queue et tous les vols internationaux sur une autre. Pour ce cas d'usage vous pouvez utiliser facilement l'exchange `direct`  avec deux bindings - une ave la clé de routage `domestic` et l'autre avec la clé de routage `international`. Cependant comme le système évolue vous réalisez que vous voulez router les même message à d'autre queues en vous basant sur d'autres dimensions comme l'aéroport ou la compagnie aérienne.

* Une manière de le faire serait de publier le message plusieurs fois- chaque fois en utilisant une clé de routage différente. Cela créé clairement du trafic superflu. Une meilleur manière de le faire serait d'utiliser l'exchange `topic` et de croiser les clés de routage. Voyons comment cela fonctionne.

## Objectif

* Router les message à des queues spécifiques en utilisant une correspondance de modèle sur les clés de routage.

## Créer l'exchange 

* Déclarer un nouvel exchange `topic`:

```
rabbitmqadmin declare exchange name=flights type=topic
```

* Les echanges `topic` permettent de faire correspondre des queues en utlisant une correspondance de modèle sur des clés de routage. La structure d'une clé de routage doit être des mots séparés par des points. Pour reprendre notre exemple d'applications sur les vols une clé de routage devrait ressembler à quelque chose comme ça : `international.lax.delta`. 
* Vous pouvez faire correspondre les queues à cet exchange en suivant les manière décrites ci-dessous :

### Correspondance sur la clé complète

```
rabbitmqadmin declare queue name=boring
rabbitmqadmin declare binding source=flights destination=boring routing_key="international.lax.delta"
```

* Ce type de binding fonctionne de la même manière qu'un binding standard sur un exchange `direct` et n'est pas si intéressant.

### Substituer des partie de la clé avec '*'

```
rabbitmqadmin declare queue name=lax
rabbitmqadmin declare binding source=flights destination=lax routing_key="*.lax.*"
```

* Ce binding assurera que seulement les vols  **LAX** sont routé vers la queue `lax`.

### Substituer des parties de la clé avec '#'

```
rabbitmqadmin declare queue name=international
rabbitmqadmin declare binding source=flights destination=international routing_key="international.#"
```

* Ce binding assurera que seul les vols **international** sont routé sur la queue `international`. Noter que contrairement à `*`, le caractère `#` peut substituer plus qu'un seul mot, comme vous pouvez le voir dans l'exemple précédent où les section  _airport_ et _airline_ ont été remplacé par `#`.

* Comme pour l'exchange `direct` vous pouvez faire correspondre plusieurs queues en utilisant la même clé de routage ou une queu peut utiliser plusieurs clés de routage

## Exercice

1. Créer une nouvelle queue (appelée `lax_delta`) et la faire correspondre à l'exchange `flights` qui recevra tous les vols Delta - domestiques et internationaux - qui arrive à LAX.
1. Envoyer un message (avec le contenu de votre choix) qui arrive dans la queue `lax_delta` mais **PAS** dans la queue `international`.

## Conclusion

* Les exchanges `topic` diffère des exchanges `direct` car on peut faire correspondre des queues en utilisant un modèles de valeurs de clés au lieu de valeurs fixe, cela permet le routage des messages multi-dimensionnel.