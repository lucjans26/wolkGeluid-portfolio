## 1. Learning outcome
Besides functionality, you develop the architecture of enterprise software based on quality attributes. You especially consider attributes most relevant to enterprise contexts with high volume data and events. You design your architecture with future adaptation in mind. Your development environment supports this by being able to independently deploy and monitor the running parts of your application.

## 2. Non-Functionals
Unlike Functional requirements, Non-functional Requirements (NFR) focus mostly on the operation of a system rather than specific behaviour or functionalities. Accounting for implementation of NFR's need t be done during the system architecture design phase because of the architectural significance. My NFR's are as follows:

- **Speed**: All requests are handled within 2 seconds
- **Security**: All requests are secured using SSL
- **Security**: Passes quality gate in Sonarclould
- **Security**: Tested by Acunetix and issues mitigated
- **Reliability**: 96% uptime, 2.5% unexpected downtime, 1.5% expected downtime
- **Closed Source**: The project will not be freely available on the version control service
- **Data Integrity**: Personal data will be stored according to GDPR regulations

## 2. Architecture
As mentioned before, designing a well thought out architecture is essential to make sure NFR's can be realised. Therefore the following design has been created:

![image](https://user-images.githubusercontent.com/46562627/203989946-04e4560a-954f-4785-bc89-27666753e0d1.png)


## 3. Messaging
To make sure all the microservices are reliably and asynchrously connected messaging needs to be introduced to the mix. This also makes sure that the load can be evenly spread and that requests are never lost internally when for example a microservice (temporarily) goes down. Furthermore a message broker makes it possible to have just a singular point of communication to the microservices because the services can "listen" to just the topics that are essential to them. This also brings with it the benefit that the architecture is easily expendable.

### 3.1 Example concept (Authentication)
On login a message is sent to the bus on the auth topic. All of the services listen to this topic on the main exchange. The message contains the userId, the type of action that is happening, and the personal access token. When this message is received by a listener, the body is rad and the according function is carried out. In this case registering the access token into the sevice's database.

### 3.2 Implementation

#### Producer
A special function for publishing a message is published as this has may be used in multiple business layers. The Method accepts a topic to which the message is sent and the payload that's sent. The exchange is created by the publisher if it doesn't exist yet.

```php
public static function publish($topic, $pload)
    {
        //php artisan rabbitmq:my-consumer --queue='user_queue' --exchange='main_exchange'
        $rabbitMQ = app('rabbitmq');
        $routingKey = $topic; // The key used by the consumer

        // The exchange (name) used by the consumer
        $exchange = new RabbitMQExchange('main_exchange', ['declare' => true]);

        $contents = $pload;

        $message = new RabbitMQMessage($contents);
        $message->setExchange($exchange);

        $rabbitMQ->publisher()->publish(
            $message,
            $routingKey
        );

        return ['message' => "Published {$contents}"];
    }
```

#### Consumer
The consumer sees incoming messages and redirects the payload to the correct method for handling dependant on the 'action' declared inside the message`. The consumer creates the queue if it doesn't exist yet as soon as the listener is started. The listener is a seperate script that runs and is overseen by [Supervisor](http://supervisord.org/).

```php
public function handle()
    {
        $rabbitMQ = app('rabbitmq');
        $messageConsumer = new RabbitMQGenericMessageConsumer(
            function (RabbitMQIncomingMessage $message) {
                $pload = json_decode($message->getStream(), true);
                switch ($pload['action']) {
                    case 'login':
                        AuthTrait::registerToken($pload['token']);
                        break;
                    case 'logout':
                        AuthTrait::deleteTokens($pload['user_id']);
                        break;
                    default:
                        break;
                }
                $message->getDelivery()->acknowledge();
            },
            $this, // Scope the closure to the command
        );

        $routingKey = 'user';
        $queue = new RabbitMQQueue('artist_queue', ['declare' => true]);
        $exchange = new RabbitMQExchange($this->option('exchange') ?? '', ['declare' => true]);

        $messageConsumer
            ->setExchange($exchange)
            ->setQueue($queue);

        $rabbitMQ->consumer()->consume($messageConsumer, $routingKey);
    }
```

### 3.3 Testing
In [this video](https://drive.google.com/file/d/15o4RT-kqFTT3HWd8W-6IMcL8IFLwLDAX/view?usp=sharing) you can see what happens during a longin. That a user is created in the auth service and that a message is produced. This message is than used by the microservices to register the accesstoken internally to authenticate withing the microservice without them needing thier own authentication methods or connection to the authService database.

## 4. Monitoring

## 5. Reflection
