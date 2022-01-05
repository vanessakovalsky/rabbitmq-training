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

![RabbitMQ - Simple Queue](https://www.rabbitmq.com/img/tutorials/python-one.png)

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
package org.springframework.amqp.demorabbitmq;

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


## Configurer notre application

* Renommer le fichier application.properties en application.yml et ajouter le contenu suivant : 
```yaml
spring:
  profiles:
    active: usage_message

logging:
  level:
    org: ERROR

demorabbitmq:
  client:
    duration: 10000
```
* Créer un package *exercise*, ajouter le fichier *ExerciseConfig.java* et ajouter le code suivant :
```java
// Sender
package org.springframework.amqp.demorabbitmq.exercise;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Profile({"exercise","hello-world"})
@Configuration
public class ExerciseConfig {

    @Bean
    public Queue hello() {
        return new Queue("hello");
    }

    @Profile("receiver")
    @Bean
    public ExerciseReceiver receiver() {
        return new ExerciseReceiver();
    }

    @Profile("sender")
    @Bean
    public ExerciseSender sender() {
        return new ExerciseSender();
    }
}

```
* Nous ajoutons dans la classe *DemoRabbitmqApplication*  les imports nécessaires pour utiliser la ligne de commande ainsi que l'appel à notre classe via la ligne de commande
```java
[...]
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Profile;
import org.springframework.scheduling.annotation.EnableScheduling;
[...]

    @Profile("usage_message")
    @Bean
    public CommandLineRunner usage() {
        return args -> {
            System.out.println("This app uses Spring Profiles to
                control its behavior.\n");
            System.out.println("Sample usage: java -jar
                demorabbitmq.jar
                --spring.profiles.active=hello-world,sender");
        };
    }

    @Profile("!usage_message")
    @Bean
    public CommandLineRunner exercuse() {
        return new DemoRabbitMQExerciseRunner();
    }
```
* Puis nous définissons la classe *DemoRabbitMQExerciseRunner* comme suit : 
```java
package org.springframework.amqp.demorabbitmq.exercise;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.ConfigurableApplicationContext;

public class DemoRabbitMQExerciseRunner implements CommandLineRunner {

    @Value("${demorabbitmq.client.duration:0}")
    private int duration;

    @Autowired
    private ConfigurableApplicationContext ctx;

    @Override
    public void run(String... arg0) throws Exception {
        System.out.println("Ready ... running for " + duration + "ms");
        Thread.sleep(duration);
        ctx.close();
    }
}
```
* Notre application est prête pour que l'on ajoute l'envoi d'un message

## Envoyer un message
* Nous allons définir le contenu de la classe d'envoi *ExerciseSender* qui se chargera d'envoyer le message à RabbitMQ : 
```java
// Sender
package org.springframework.amqp.demorabbitmq.exercise;

import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;

public class ExerciseSender {

    @Autowired
    private RabbitTemplate template;

    @Autowired
    private Queue queue;

    @Scheduled(fixedDelay = 1000, initialDelay = 500)
    public void send() {
        String message = "Hello World!";
        this.template.convertAndSend(queue.getName(), message);
        System.out.println(" [x] Sent '" + message + "'");
    }
}
```
* Nous auto-chargeons la queue qui a été configuré dans ExerciseConfig et nous utilisons le modèle *RabbitTemplate* pour charger le client RabbitMQ. 
* Dans la méthode *send* nous utilisons la méthode du modèle *convertAndSend* pour envoyer notre message à la queue choisie.
* Votre classe est maintenant prête à envoyer un message. 

## Recevoir le message

* Nous allons maintenant définir la classe qui se charge de receptionner le message et de l'afficher
```java
package org.springframework.amqp.demorabbitmq.exercise;

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;

@RabbitListener(queues = "hello")
public class ExerciseReceiver {

    @RabbitHandler
    public void receive(String in) {
        System.out.println(" [x] Received '" + in + "'");
    }
}
```
* Nous utilisons l'annotaiton *@RabbitListerner* qui permet d'utiliser le listener du client et lui passons le nom de la queue que nous souhaitons suivre.
* Puis nous annotons notre méthode *receive* avec l'annotation *@RabbitHandler* ce qui permet de récupérer le payload envoyé à la queue que nous utilisons pour afficher le contenu du message.

## Lancer l'application

* Il ne nous reste plus qu'à exécuter notre application.
* Commençons par la compiler avec maven
```
./mvnw clean package
```
* Puis nous lançons le recepteur avec la commande : 
```
java -jar target/demorabbitmq.jar --spring.profiles.active=hello-world,receiver
```
* Enfin, dans un autre terminal, nous envoyons le message 
```
java -jar target/demorabbitmq.jar --spring.profiles.active=hello-world,sender
```
* Dans le terminal du récepteur le message devrait s'afficher.