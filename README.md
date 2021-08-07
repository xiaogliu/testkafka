Please see this article to find more detail info [Easy Java Springboot and Apache Kafka Setup on macOS](https://levelup.gitconnected.com/easy-java-springboot-apache-kafka-setup-on-macos-ceb481e167f8)

![https://miro.medium.com/max/700/0*-CvUZ6GRTWQ_KZ0x.jpg](https://miro.medium.com/max/700/0*-CvUZ6GRTWQ_KZ0x.jpg)

Image source from Cloudkarafka

# **Requirements**

- [Homebrew](https://brew.sh/)
- Testing with [Postman](https://www.postman.com/). To learn more, you can watch this [video](https://www.youtube.com/watch?v=t5n07Ybz7yI&feature=emb_logo).
- **[Understanding the core concept of Kafka](https://www.youtube.com/watch?v=udnX21__SuU)**

# **Installment**

1. **Install Java 8**

```
$ brew tap adoptopenjdk/openjdk
$ brew cask install adoptopenjdk8
```

**2. Install Kafka**

```
$ brew install kafka
```

3. Open **the 1st terminal** to start Zookeeper

```
$ zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties
```

4. Open **the 2nd terminal** to start Kafka

```
$ kafka-server-start /usr/local/etc/kafka/server.properties
```

5. Open **the 3rd terminal** to create a Kafka topic

```
$ kafka-topics --bootstrap-server localhost:9092 --topic<enter-a-topic> --create --partitions 1 --replication-factor 1
```

# **Create a Maven project with Springboot**

1. Open [start.spring.io](https://start.spring.io/)
2. Choose **Spring Web** & **Spring for Apache Kafka** dependencies

![https://miro.medium.com/max/700/1*XvNJ-Qs8SqCMqS0QBXgBaA.png](https://miro.medium.com/max/700/1*XvNJ-Qs8SqCMqS0QBXgBaA.png)

3. Open your project in IDE (I prefer to use IntelliJ)

# **Configure Kafka Producer and Consumer**

1. Add this to `application.properties` in `src/main/resources` folder, and modify the highlights:

```
spring.datasource.url=jdbc:mysql://localhost:3306/<your-db-name>?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC
spring.datasource.username=<your-db-username>
spring.datasource.password=<your-db-password>app.topic=<same-topic-created-in-terminal>spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.groupId=<any-group-id>spring.kafka.consumer.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=<any-group-id>
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializerspring.kafka.producer.bootstrap-servers=localhost:9092
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
```

2. Open a new terminal, and create a topic:

```
kafka-topics --bootstrap-server localhost:9092 --topic<same-topic-in-application.properties> --create --partitions 1 --replication-factor 1
```

# **Initialize Kafka Producer & Receiver Service**

1. Inside the package `com.example`, create these folders: `$ mkdir controller model repository service`
2. Create Producer and Receiver Services: `$ touch service/KafkaConsumer.java service/KafkaProducer.java`
3. Add this to **KafkaConsumer:**

```
package <enter-your-package-here>;import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;import java.util.ArrayList;
import java.util.List;@Service
public class KafkaConsumer {    public static List<String> messages = new ArrayList<>();
    private final static String topic ="<same-topic-in-resources>";
    private final static String groupId ="<same-group-id>";    @KafkaListener(topics = topic, groupId = groupId)
    public void listen(String message) {
        messages.add(message);
    }
}
```

4. Add this to KafkaProducer:

```
package <enter-your-package-here>;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class KafkaProducer {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @Value("${app.topic}")
    private String topic;

    public void produce(String message) {
        kafkaTemplate.send(topic, message);
    }

}
```

# **Create Kafka API**

1. Create a Kafka controller file: `$ touch controller/kafkaController.java` and add this:

```
package server.controller;

import org.springframework.web.bind.annotation.*;

import server.service.KafkaConsumer;
import server.service.KafkaProducer;

import org.springframework.beans.factory.annotation.Autowired;

import java.util.List;

@RestController
public class KafkaController {
  @Autowired
  private KafkaConsumer consumer;

  @Autowired
  private KafkaProducer producer;

  @PostMapping("/send")
  public void send(@RequestBody String data) {
    producer.produce(data);
  }
  @GetMapping("/receive")
  public List<String> receive() {
    return consumer.messages;
  }

  public KafkaConsumer getConsumer() {
    return consumer;
  }

  public KafkaProducer getProducer() {
    return producer;
  }

  public void setConsumer(KafkaConsumer consumer) {
    this.consumer = consumer;
  }

  public void setProducer(KafkaProducer producer) {
    this.producer = producer;
  }
}
```

*NOTE: There are also models and repository, which you use to configure with your database, but we don’t need it in this tutorial.*

![https://miro.medium.com/max/421/1*FZgx3Rg62Xw4UE5-YlVZSg.png](https://miro.medium.com/max/421/1*FZgx3Rg62Xw4UE5-YlVZSg.png)

Final Project Structure

# **Testing**

There are 2 ways to send the message from the producer: Postman & Terminal. I’ll show you both:

1. Open the 3rd terminal that you used to create the topic earlier

```
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic<enter-your-topic> --from-beginning
```

## **2. Postman**

2.1 Use **POST/** method and enter **localhost:8080/send** with the JSON body:

```
{
   "message": "Hello from Postman"
}
```

2.2 Open the **4th terminal,** and type:

```
$ kafka-console-producer --broker-list localhost:9092 --topic<enter-your-topic>
> Hello from the terminal
```

3. Check the terminal, you will see the message that you sent via postman and terminal **in real-time.**

![https://miro.medium.com/max/1000/1*3T6hwyuejLCpiF6rtLWsAg.png](https://miro.medium.com/max/1000/1*3T6hwyuejLCpiF6rtLWsAg.png)

# **Well done!**

Now, you’re able to set up your own Java Spring and Apache Kafka with confidence!
