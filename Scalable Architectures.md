## 1. Learning outcome
Besides functionality, you develop the architecture of enterprise software based on quality attributes. You especially consider attributes most relevant to enterprise contexts with high volume data and events. You design your architecture with future adaptation in mind. Your development environment supports this by being able to independently deploy and monitor the running parts of your application.

## 2. Non-Functionals
Unlike Functional requirements, Non-functional Requirements (NFR) focus mostly on the operation of a system rather than specific behaviour or functionalities. Accounting for implementation of NFR's need t be done during the system architecture design phase because of the architectural significance. My NFR's are as follows:

### 2.1 Definition
- **Speed**: All requests are handled within 2 seconds
- **Security**: Passes quality gate in Sonarclould
- **Security**: Tested by Acunetix and issues mitigated
- **Reliability**: 96% uptime, 2.5% unexpected downtime, 1.5% expected downtime
- **Closed Source**: The project will not be freely available on the version control service
- **Data Integrity**: Personal data will be stored according to GDPR regulations

### 2.2 Validation

- **Speed**: All requests are handled within 2 seconds: Done through load testing using K6 on a Kubernetes deployment of the system
- **Security**: Passes quality gate in Sonarclould: Can be checked withing Sonarcloud
- **Security**: Tested by Acunetix and issues mitigated: Proof though Mitigation raport
- **Reliability**: 96% uptime, 2.5% unexpected downtime, 1.5% expected downtime: (group project) Explain uptime of Azure Kubernetes Kluster
- **Closed Source**: The project will not be freely available on the version control service: Explain private repositories
- **Data Integrity**: Personal data will be stored according to GDPR regulations: Proof through right to be forgotten

### 2.3 "Implementation"

#### 2.3.1 Speed

#### 2.3.2 Security (Sonarcloud)
The quality gate can be used to help ensure that codebases meet certain security standards. For example, the quality gate could be configured to require that codebases have a certain level of test coverage, to ensure that all relevant code paths are being tested and that the code is properly tested for security vulnerabilities. It could also be configured to require that codebases have a certain level of security hardening, such as requiring the use of secure coding practices and the inclusion of security headers.
![image](https://user-images.githubusercontent.com/46562627/210672819-b1131114-bb69-4419-8745-0c267cf48c94.png)


#### 2.3.3 Security (Acunetix)
#### 2.3.4 Reliability
The two mentioned types of downtime have different causes and fixes.

Expected downtime is downtime needed for maintenance, migrations, etc. This is downtime that can be controlled. By setting up zero downtime deployments on the kubernetes server we can make sure that the downtime for these things are kept to a minimum and the downtime is essentially 0. Migrations might still need some downtime, but migrations don't happen on the daily. Were this to be needed, the service could be offline for an entire day in a year while the requirement would still be met.

Unexpected downtime is downtime caused by issues like traffic overload or cluster downtime. By hosting the service on an Azure Kubernetes Cluster these problems kan be mitigated for the most part. Azure garuenteed [%99.95](https://learn.microsoft.com/en-us/azure/aks/uptime-sla) uptime SLA. Besides that hosting the cluster on a Azure provides us with other benefits:

- Redundancy and failover: AKS clusters are designed to be highly available and redundant, with multiple nodes and infrastructure components that can automatically failover in the event of a failure or outage.
- Monitoring and alerting: AKS clusters are monitored by Azure and can send alerts if there are any issues or problems.
- Automatic scaling: AKS clusters can automatically scale up or down based on workload demand, which can help to ensure that applications have the resources they need to function properly.
- Maintenance and updates: AKS clusters are managed by Azure and receive regular maintenance and updates to ensure that they are running the latest software and hardware.
By using these redundancies and capabilities we can make sure that unexpected downtime is kept to a minumum.

#### 2.3.5 Closed Source
After the any access by teachers for school purposes is not needed anymore the repositories of the services will be set to private. This makes sure that source code is not leaked to prevent copycats and makes the application more secure by hiding any possible flaws from people with malicious intent.

#### 2.3.6 Data Integrity
[Implementing the right to be forgotten](https://github.com/lucjans26/wolkGeluid-portfolio/blob/main/Security%20by%20design.md#22-implementation) is a great proof of concept for data storage in accordance with GDPR regulations because it shows that it is possible to effectively delete personal data when requested by the user. This demonstrates that data controllers in multiple services are able to comply with the GDPR's requirement to only process personal data in a way that is necessary.

Many more GDPR laws could and should be implemented to ensure a secure organisation and system with protected user data if more time and resources would be available.


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

## 4. Gateway
A gateway is a software pattern that sits in front of a group of microservices to faciliate requests and delivery of data and services. It functions as a single entrypoint to a complete architecture and apply policies to determine abilities and behaviour  for the group of microservices to determine avalibility and behaviour.

### 4.1 Setup
For my architecture I chose to create an Ocelot (.Net) gateway. Creating a basic gateway using Ocelot is fairly mundane. After creating a new API Project in visual studio we can install the Ocelot NuGet package.

![image](https://user-images.githubusercontent.com/46562627/204158626-4ea5bb4f-8dc2-4f77-9909-0c7874169d25.png)

After this we can add the necessary methods in the Program.cs. These methods handle starting the gateway correctly. After that we can setup the ocelot.json routing file.

![image](https://user-images.githubusercontent.com/46562627/204159114-4611b4b8-52c9-4f80-8e7b-f23aed6541ec.png)

In this example we can see the setup for a single endpoint in the song service. In this case the endpoint a basically identical. Everything related to "downstream" handles where the request will eventually go to. In the example we can see that the request wil go to localhost, port 8080, to the /api.auth/google/redirect endpoint. Any request that goes to the gateway endpoint (upstream) /api/auth/google will be sent to the downstream. The gateway accepts just the get method in this case, but the array may contain any method type.

### 4.1 Test
To see the functionality of the gateway i've made [this video](https://drive.google.com/file/d/12soJ4q70z8mJMzntvOjRYcOfjrzZoVZI/view?usp=sharing). In the video you can see that all my microservices and the gateway are running. The gateway is running on port 5000 as configured in the settings, and when making a request to the gateway I get a response from the authentication service on port 8080 where I perform a login.

## 5. Monitoring
Monitoring is the practice of tracking the performance and availability of a system. It involves collecting data about the system (CPU usage, response times, logs, availibility), analyzing that data, and using it to identify any issues or activities that may indicate problems. It is important for multiple reasons, it:
- Helps ensure the system is performing well and meets the needs of the users
- Can help identify and troubleshoot issues
- Provide insight into how the system is being used to optimize it's performance and scalability

### 5.1 Setup
The monitoring system consists out of three seperate parts; the data "source" ([InfluxDB](https://www.influxdata.com/)), the data collection agent ([Telegraf](https://www.influxdata.com/time-series-platform/telegraf/)), and the visualisation tool ([Grafana](https://grafana.com/grafana/)).
InfluxDB is a time series database that can be used to store and query large amounts of [time-series data](https://www.influxdata.com/what-is-time-series-data/). Telegraf is a data collection agent that can be used to collect metrics from various sources and write them to InfluxDB. Grafana is a visualization tool that can be used to create dashboards and graphs of the data stored in InfluxDB.

![image](https://user-images.githubusercontent.com/46562627/210437384-1883a073-51ce-4423-89ee-147b0174328c.png)

Firstly the InfluxDB is set up so that the data can be saved somewhere. This can easily be done with the InfluxDB docker image. 
Next Telegraf needs to be set up. This too can be done using the proper docker image. However Telegraf does require some configuring. This mostly comes down to setting up the input (rabbitMQ) and the output (InfluxDB). The conatiner names can be used as the host names because they exist in the same docker network.

```env
[[outputs.influxdb]]
  urls = ["http://influxdb-1:8086"]
  database = "influx"
  timeout = "5s"
  username = "admin"
  password = "admin"
  
[[inputs.rabbitmq]]
  url = "http://rabbitmq:15672"
  username = "guest"
  password = "guest"
```

After this the most important part is to save the file using UTF-8 (non BOM) encoding or else the config file will not be valid....(This took me way too long to figure out)

Finally we can simple deploy the Grafana image where we can connect the data source (InfluxDB).

![image](https://user-images.githubusercontent.com/46562627/210438024-136b2b60-6a91-4ce1-93e5-d3e73de342dc.png)

Of course and Azure monitor data source could also be configured. Unfortunately this is not possible with the student subscription that is provided by school. Were this possible, it could be implemented the same way rabbitMQ will be.

### 5.2 Implementation
It is important to monitor essetial metrics of a (sub)system. Monitoring metrics that aren't important wil result in a crowded screen that makes it hard to spot real problems. Therefore I chose to monitor the following metrics on the rabbitMQ deployment; Node status, Unacknowlegded (queued) messages, available system memory. 

A few queued messages are to be expected. after all this is the point of using a message queue. But messages piling up mean that a service is not running as expected. In the following demonstration I paused one of the services so the messages are nog consumed from the queue. The metric is set up in a way that means that 0 - 4 queued messages result in a green indicator, 5 - 7 in an orange indicator, and 8+ result in a red indicator.

![image](https://user-images.githubusercontent.com/46562627/210440052-9305cb13-69ff-4289-9dcd-e80938d2dea1.png)

In the next demonstration I stopped the rabbitMQ container. This means that the node is essentially down. The node status metric is set up to turn red when the node is not running which result in the following visual:
![image](https://user-images.githubusercontent.com/46562627/210440797-e3b416b3-0842-42cb-9167-e126ff97ee89.png)

The available memory metric gives an indication on how the system is handeling the load. The guage gives a quick insight into the current status of the system. However at times it may be relevant to check back in time on what happened. For this a time series can added.

Many more metrics and systems can be monitored, and the customisation is almost endless. However the form of monitoring and the setup should fit the needs of the system or application to be monitored.

## 6. Reflection
