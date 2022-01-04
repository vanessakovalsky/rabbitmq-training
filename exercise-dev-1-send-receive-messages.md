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

* Nous avons besoin d'ajouter des classes à importer pour envoyer des messages.
* Créer un fichier *Send.java* et ajouter les dépendance suivantes :
```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
```
* Puis créer la classe et définir un nom pour la queue de message :
```java
public class Send {
  private final static String QUEUE_NAME = "bonjour";
  public static void main(String[] argv) throws Exception {
      ...
  }
}
```
* Puis nous essayos de nous connecter sur le serveur local : 
```java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
try (Connection connection = factory.newConnection();
     Channel channel = connection.createChannel()) {

}
```
* L'objet connexion initialise la connexion socket et s'occupe de la negociation de version du protocole et de l'authentification. 
* Ensuite nous avons dfinis un objet channel, qui est l'objet principal de cette API. Il n'est pas nécessaire de fermer la connexion et le channel, puisque les deux classes utilisée utilie *java.io.Closeable* qui s'en charge 
* Pour envoyer un message, nous devons déclarer une queue qui fait l'envoi pour nous, puis nous pouvons publier un message sur la queue. 
* Ajouter le code suivant dans le bloc *try* :
```java
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
String message = "Bonjour à tous !";
channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
System.out.println(" [x] Sent '" + message + "'");
```
* La déclaration de queue ne créé la queu que si celle-ci n'existe pas déjà. Le contenu du message est un tableau de bytes vous permettant d'encoder ce que vous souhaiter. 
* Votre message est maintenant envoyé (en cas d'erreur, vérifier l'espace disque disponible, et les logs pour trouver ce qui ne fonctionne pas.)

## Recevoir le message

* Nous allons maintenant définir une classe qui va récupérer le message dans la queue.
* Pour cela créer un fichier *Recv.java* avec les imports suivants : 
```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;
```
* Il s'agit des mêmes imports que pour l'envoi, à l'execption de l'interface *DeliverCallback* qui permet de mettre dans le tampon les messages qui nous sont envoyés par le serveur
* Comme pour l'envoi, nous ouvrons une connexion et un chanel et déclarons la queue à utiliser pour recevoir les messages. 
```java
public class Recv {

  private final static String QUEUE_NAME = "bonjour";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.queueDeclare(QUEUE_NAME, false, false, false, null);
    System.out.println(" [*] En attente de message. Pour quitter appuyer sur CTRL+C");

  }
}
```
* La queue est également redéclarer ici, cela permet d'éviter si le receveur est démarré avant l'envoyeur de ne pas essayer de recevoir des messages d'une queue qui n'existe pas encore.
* Nous n'utilisons pas de bloc *try* car en l'absence de message à recevoir cela fermerait la connexion et le channel et quitterait le programme. Ce qui serait contre-productif alors que notre recepteur attend des messages.
* Nous allons maintenant  demander au serveur de nous envoyer les messages depuis la queue. Puisque l'envoie des message est fait de manière asynchrone, nous fournissons un *callback* sous la forme d'un objet qui stockera les messages en attendant que nous les utilisisons. 
```java
DeliverCallback deliverCallback = (consumerTag, delivery) -> {
    String message = new String(delivery.getBody(), "UTF-8");
    System.out.println(" [x] Received '" + message + "'");
};
channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
```
* Notre code est prêt

## Exécuter notre code

* Pour executer notre code commençons par le compiler : 
```
javac -cp amqp-client-5.7.1.jar Send.java Recv.java
```
* Puis pour exécuter le receveur : 
```
java -cp .:amqp-client-5.7.1.jar:slf4j-api-1.7.26.jar:slf4j-simple-1.7.26.jar Recv
```
* Et enfin la classe d'envoi (à éxecuter dans un autre terminal) : 
```
java -cp .:amqp-client-5.7.1.jar:slf4j-api-1.7.26.jar:slf4j-simple-1.7.26.jar Send
```
* Vous devriez alors voir dans le terminal qui exécute le recepteur le message défini s'afficher.