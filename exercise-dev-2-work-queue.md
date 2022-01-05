# Distribuer le travail avec les work queue

## Objectifs

Cet exercice a pour objectif
* de mettre en place une work queue qui permet de distribuer le travail à différent consommateurs.

## Présentation

* RabbitMQ permet d'envoyer des messages dans des queues. Celle-ci peuvent servir simplement à garder les messages le temps qu'un recepteur les traite comme dans notre premier exercice.
* Il arrive aussi que l'on est besoin que les messages doivent être distribués à plusieurs client (consommateur) afin d'être traité en parallèlle. RabbitMQ propose alors des queues de travail (Work Queues ou Task Queues) qui permet cela. 

![Work Queue ou Task Queue](https://www.rabbitmq.com/img/tutorials/python-two.png)

* Pour cet exercice nous allons reprendre le package demorabbitmq.exercise que nous avons créé et le modifier

* Au niveau de la classe de configuration (ExerciseConfig) ajouter un deuxième *receiver*

```java
[...]
@Profile("receiver")
    private static class ReceiverConfig {

        @Bean
        public ExerciseReceiver receiver1() {
            return new ExerciseReceiver(1);
        }

        @Bean
        public ExerciseReceiver receiver2() {
            return new ExerciseReceiver(2);
        }
    }
[...]
```

## Envoi des messages

* Afin d'identifier si le message est une tâche longue et doit donc être séparé en plusieurs messages, nous ajoutons un point au message. 
* Pour cela on modifie la classe *ExerciseSender* comme suit : 
 ```java
[...]
import java.util.concurrent.atomic.AtomicInteger;
[...]
    AtomicInteger dots = new AtomicInteger(0);

    AtomicInteger count = new AtomicInteger(0);

    @Scheduled(fixedDelay = 1000, initialDelay = 500)
    public void send() {
        StringBuilder builder = new StringBuilder("Hello");
        // On incrémente notre dots et lorsqu'il atteint 4 on le remet à 1
        if (dots.incrementAndGet() == 4) {
            dots.set(1);
        }
        // On ajoute autant de points à notre chaine que la valeur de dots
        for (int i = 0; i < dots.get(); i++) {
            builder.append('.');
        }
        // On ajoute la valeur de count à la chaine
        builder.append(count.incrementAndGet());
        // On transforme notre objet en chaine de caractère
        String message = builder.toString();
        // On convertit en message et on l'envoit via le client RabbitMQ
        template.convertAndSend(queue.getName(), message);
        System.out.println(" [x] Sent '" + message + "'");
    }
[...]
```

## Recevoir les messages

* Notre récepteur simule une longueur arbitraire où le nombre de points est transformer en nombre de seconde pour traiter la tâche. 
* Voici son code
 ```java
[...]
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.util.StopWatch;

@RabbitListener(queues = "hello")
public class ExerciseReceiver {

    private final int instance;

    public ExerciseReceiver(int i) {
        this.instance = i;
    }

    @RabbitHandler
    public void receive(String in) throws InterruptedException {
        StopWatch watch = new StopWatch();
        watch.start();
        System.out.println("instance " + this.instance +
            " [x] Received '" + in + "'");
        doWork(in);
        watch.stop();
        System.out.println("instance " + this.instance +
            " [x] Done in " + watch.getTotalTimeSeconds() + "s");
    }

    private void doWork(String in) throws InterruptedException {
        for (char ch : in.toCharArray()) {
            if (ch == '.') {
                Thread.sleep(1000);
            }
        }
    }
}
```
* Nous utilisons toujours les annotations *@RabbitListener* sur la queue *hello* et *@RabbitHandler* pour traiter le message reçu. 
* Nous affichons le message et la durée en seconde du traitement de celui-ci (avec notre faux calcul).

## Exécuter le code 
* Nous compilons et exécutons le code (comme dans l'exercice précédent dans deux terminaux différents)
 ```
./mvnw clean package

# shell 1
java -jar target/demorabbitmq.jar --spring.profiles.active=work-queues,receiver
# shell 2
java -jar target/demorabbitmq.jar --spring.profiles.active=work-queues,sender
```
* La sortie du récepteur ressemble alors à celle-ci : 
```
Ready ... running for 10000ms
 [x] Sent 'Hello.1'
 [x] Sent 'Hello..2'
 [x] Sent 'Hello...3'
 [x] Sent 'Hello.4'
 [x] Sent 'Hello..5'
 [x] Sent 'Hello...6'
 [x] Sent 'Hello.7'
 [x] Sent 'Hello..8'
 [x] Sent 'Hello...9'
 [x] Sent 'Hello.10'
```
* Et la sortie du worker : 
```
Ready ... running for 10000ms
instance 1 [x] Received 'Hello.1'
instance 2 [x] Received 'Hello..2'
instance 1 [x] Done in 1.001s
instance 1 [x] Received 'Hello...3'
instance 2 [x] Done in 2.004s
instance 2 [x] Received 'Hello.4'
instance 2 [x] Done in 1.0s
instance 2 [x] Received 'Hello..5'
```

=> Vous savez maintenant distribué des taches entre les workers