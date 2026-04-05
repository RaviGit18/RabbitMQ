# RabbitMQ Spring Boot Application

A modern Spring Boot application demonstrating RabbitMQ integration with Spring AMQP, featuring RESTful endpoints and message-driven architecture.

## Overview

This module showcases how to integrate RabbitMQ with Spring Boot using Spring AMQP, providing a clean and idiomatic approach to messaging with minimal configuration.

## Features

### Spring Boot Integration
- **Auto-configuration** for RabbitMQ connection and template
- **Spring AMQP** abstractions for messaging
- **RESTful API** endpoints for publishing messages
- **Message Listeners** with `@RabbitListener` annotations
- **JSON Message Serialization** with Jackson

### Key Components
- `RabbitMqApplication.java` - Spring Boot main application
- `RabbitMQConsumer.java` - Message consumer implementation
- `TestController.java` - REST controller for message publishing
- `Person.java` - Message model/entity
- `application.properties` - Configuration

## Technology Stack

- **Spring Boot 2.2.2.RELEASE**
- **Spring AMQP** (RabbitMQ integration)
- **Spring Web** (REST endpoints)
- **Java 8**
- **Maven**

## Prerequisites

- Java 8 or higher
- Maven 3.6 or higher
- RabbitMQ server running on localhost:5672

## Quick Start

### 1. Start RabbitMQ Server
```bash
# Using Docker
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

### 2. Build and Run the Application
```bash
# Build
mvn clean package

# Run
java -jar target/RabbitMQ-0.0.1-SNAPSHOT.jar

# Or using Maven
mvn spring-boot:run
```

### 3. Test the Application

The application will start on `http://localhost:8080`

#### Send a Message via REST API
```bash
# Send a person message
curl -X POST http://localhost:8080/send \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","age":30,"city":"New York"}'
```

#### Check Application Health
```bash
curl http://localhost:8080/actuator/health
```

## Configuration

### application.properties
```properties
# RabbitMQ Configuration
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

# Server Configuration
server.port=8080
```

### Custom RabbitMQ Configuration
You can customize the RabbitMQ connection settings in `src/main/resources/application.properties`:

```properties
# Custom RabbitMQ settings
spring.rabbitmq.host=your-rabbitmq-host
spring.rabbitmq.port=5672
spring.rabbitmq.username=your-username
spring.rabbitmq.password=your-password
spring.rabbitmq.virtual-host=your-vhost

# Connection settings
spring.rabbitmq.connection-timeout=15000
spring.rabbitmq.publisher-confirm-type=correlated
spring.rabbitmq.publisher-returns=true
```

## API Endpoints

### POST /send
Publishes a Person message to RabbitMQ.

**Request:**
```json
{
  "name": "John Doe",
  "age": 30,
  "city": "New York"
}
```

**Response:**
```
Message sent successfully!
```

### GET / (Root)
Simple health check endpoint.

## Message Flow

```
HTTP Request → TestController → RabbitTemplate → RabbitMQ → RabbitMQConsumer
```

1. Client sends HTTP POST request to `/send`
2. `TestController` receives the request and creates a `Person` object
3. `RabbitTemplate` sends the message to RabbitMQ
4. `RabbitMQConsumer` receives and processes the message

## Project Structure

```
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── example/
│   │           └── demo/
│   │               ├── RabbitMqApplication.java    # Main application
│   │               ├── RabbitMQConsumer.java        # Message consumer
│   │               ├── TestController.java          # REST controller
│   │               └── Person.java                  # Message model
│   └── resources/
│       └── application.properties                  # Configuration
└── test/
    └── java/
        └── com/
            └── example/
                └── demo/
                    └── RabbitMqApplicationTests.java
```

## Key Components Explained

### RabbitMQConsumer
```java
@Component
public class RabbitMQConsumer {
    
    @RabbitListener(queues = "queue-1")
    public void receiveMessage(Person person) {
        System.out.println("Received message: " + person);
    }
}
```

### TestController
```java
@RestController
public class TestController {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @PostMapping("/send")
    public String sendMessage(@RequestBody Person person) {
        rabbitTemplate.convertAndSend("queue-1", person);
        return "Message sent successfully!";
    }
}
```

### Person Model
```java
public class Person {
    private String name;
    private int age;
    private String city;
    // getters and setters
}
```

## Advanced Configuration

### Custom RabbitTemplate
```java
@Configuration
public class RabbitConfig {
    
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(new Jackson2JsonMessageConverter());
        return template;
    }
}
```

### Exchange and Queue Configuration
```java
@Bean
public Queue queue() {
    return QueueBuilder.durable("queue-1").build();
}

@Bean
public TopicExchange exchange() {
    return new TopicExchange("exchange-1");
}

@Bean
public Binding binding(Queue queue, TopicExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange).with("routing.key");
}
```

## Testing

### Unit Tests
```bash
mvn test
```

### Integration Testing
The application includes a basic test class `RabbitMqApplicationTests.java` for context loading.

### Manual Testing
1. Start the application
2. Use curl or Postman to send messages
3. Check console output for consumed messages

## Monitoring and Management

### Spring Boot Actuator
The application includes Spring Boot Actuator for monitoring:

- Health checks: `http://localhost:8080/actuator/health`
- Application info: `http://localhost:8080/actuator/info`

### RabbitMQ Management UI
Access RabbitMQ management interface at: `http://localhost:15672`
- Username: guest
- Password: guest

## Troubleshooting

### Common Issues
1. **Connection Refused** - RabbitMQ server not running
2. **Authentication Failed** - Incorrect credentials
3. **Queue Not Found** - Queue not properly declared
4. **JSON Serialization Errors** - Invalid message format

### Debug Tips
- Check application logs for connection errors
- Verify RabbitMQ management UI shows active connections
- Ensure queue exists and is bound correctly
- Check message format in JSON

## Best Practices

1. **Use durable queues** for production
2. **Implement message acknowledgments** for reliability
3. **Handle connection failures** with retry logic
4. **Use appropriate exchange types** for your use case
5. **Monitor queue depths** to prevent memory issues

## Next Steps

- Add message acknowledgments
- Implement error handling and dead letter queues
- Add message routing with exchanges
- Implement batch processing
- Add integration tests with TestContainers
