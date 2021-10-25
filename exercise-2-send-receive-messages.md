# Exercice 2 - Envoyer et recevoir des messages

## Objectifs
Cet exercice a pour objectifs :
* d'envoyer des message avec un client Java
* de recevoir des messages avec un client Java

## Présentation

RabbitMQ est un agent de message, c'est à dire qu'il reçoit et transmet des messages.
Pour cela il s'appuie sur les 3 concepts fondamentaux suivants : 
* producteur : c'est celui qui rédige et envoi le message
* queue (ou file d'attente) : c'est l'endroit par lequel le message transite
* consommateur ou récepteur : c'est lui qui reçoit et traite le message 

!(RabbitMQ - Simple Queue)[https://www.rabbitmq.com/img/tutorials/python-one.png]

## Installation de la bibliothèque cliente Java

 * Initialiser une nouvelle application SpringBoot, a l'aide de https://start.spring.io/
 * Dans l'interface sur la partie droite choisir Spring Boot Web en tant que dépendance, puis générer et enregistrer le code. 
 * Manuellement, ajouter la dépendance suivante à votre projet :
 ```xml 
 <dependency>
  <groupId>com.rabbitmq</groupId>
  <artifactId>amqp-client</artifactId>
  <version>5.13.1</version>
</dependency>
```
* Ajouter un peu de code dans la classe, par exemple ici je rajoute une fonction hello :
```java
package com.kovalibre.demorabbitmq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class DemoRabbitmqApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoRabbitmqApplication.class, args);
	}

	@GetMapping("/hello")
	public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
		return String.format("Hello %s!", name);
	}

}
```
* Pour lancer votre projet utiliser la commande : 
```
./mvnw spring-boot:run
```
* Vous pouvez accéder à votre application sur http://localhost:8080


## Envoyer un message
