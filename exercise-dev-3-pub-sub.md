# Exercice dev 3 - Pub / Sub

## Objectifs :
Cet exercice a pour objectif :
* de publier un modèle fanout qui permet d'envoyer un message à plusieurs consomateurs. 

## Présentation

* Le type d'exchange fanout permet d'envoyer un message à plusieurs queues sans filtrer ceux-ci.
* On a alors un envoit qui ressemble à ça :
![Fanout exchange type](https://www.rabbitmq.com/img/tutorials/exchanges.png)

* Créer un nouveau package exercice.fanout avec un fichier de configuration contenant le code suivant : 
```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Profile({"fanout", "pub-sub", "publish-subscribe"})
@Configuration
public class Tut3Config {

    @Bean
    public FanoutExchange fanout() {
        return new FanoutExchange("exercise.fanout");
    }

    @Profile("receiver")
    private static class ReceiverConfig {

        @Bean
        public Queue autoDeleteQueue1() {
            return new AnonymousQueue();
        }

        @Bean
        public Queue autoDeleteQueue2() {
            return new AnonymousQueue();
        }

        @Bean
        public Binding binding1(FanoutExchange fanout,
            Queue autoDeleteQueue1) {
            return BindingBuilder.bind(autoDeleteQueue1).to(fanout);
        }

        @Bean
        public Binding binding2(FanoutExchange fanout,
            Queue autoDeleteQueue2) {
            return BindingBuilder.bind(autoDeleteQueue2).to(fanout);
        }

        @Bean
        public ExerciseFanoutReceiver receiver() {
            return new ExerciseFanoutReceiver();
        }
    }

    @Profile("sender")
    @Bean
    public ExerciseFanoutSender sender() {
        return new ExerciseFanoutSender();
    }
}
```
* Comme dans les exercices précédents nous avons : 
    * défini un profile
    * défini un Spring Bean avec la méthode *FanoutExchange*
    * instancier les classes Receiver et Sender
    * dans la classe receiver, nous avons définis 2 *AnonymousQueue* qui crée des queues éphémères et deux bindings sur ces queues.

## Envoi du message

* Créer la classe ExerciseFanoutReceiver comme dans l'exercice 2 
* Pour envoyer le message nous utilisons la même méthode *convertAndSend* que dans les précédents exercices : 
```java
@Autowired
private RabbitTemplate template;

@Autowired
private FanoutExchange fanout;   // configured in ExerciseFanoutConfig above

template.convertAndSend(fanout.getName(), "", message);
```
* La seul spécificité lors de l'envoi du message est que nous sommes obligé de passer le paramètre de clé de routage à vide (le paramètre est obligatoire car décrit comme tel dans le protocle AMQP)

## Reception du message dans les queues

* Comme expliqué nous avons créé deux queues éphémères