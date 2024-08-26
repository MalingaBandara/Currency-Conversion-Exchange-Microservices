
# Currency Conversion and Exchange API with Microservices

This project leverages Spring Boot, Microservices architecture, Cloud computing, and Docker for implementing a Currency Conversion and Exchange API. It comprises several microservices designed to handle currency-related operations efficiently.


## Project Overview

This project leverages Spring Boot, microservices architecture, cloud computing, and Docker for implementing a currency conversion and exchange API. The application is composed of multiple microservices that handle different aspects of the currency conversion and exchange process, including service discovery, load balancing, and request routing. It demonstrates a robust, scalable, and resilient microservices architecture.

## Key Features

- Currency conversion and exchange operations.
- Service discovery with Eureka for seamless microservices coordination.
- Centralized API Gateway for request routing and load balancing.
- Distributed tracing with Zipkin for monitoring inter-service communication.
- Asynchronous communication using RabbitMQ.
- Dockerized deployment for consistency across environments.

## Technologies Used

- **Spring Boot**: For rapid development of microservices.
- **Microservices Architecture**: For scalability, resilience, and maintainability.
- **Cloud Computing**: Utilized for deployment, offering scalability and accessibility.
- **Docker**: Used for containerization, ensuring consistency across various environments.
- **Eureka**: For service discovery and registration.
- **Zipkin**: Providing distributed tracing capabilities.
- **H2 Database**: Used for storing exchange rates data.
- **RabbitMQ**: For asynchronous communication between microservices.
- **Resilience4j**: Offering fault tolerance and resiliency to microservices.

## Microservices Overview

1. **Currency Conversion Microservice**: Handles currency conversion based on exchange rates.
2. **Currency Exchange Microservice**: Executes currency exchange operations.
3. **Naming Server (Eureka)**: Manages service discovery and registration.
4. **API Gateway**: Centralized entry point for routing and load balancing across microservices.

## Repository Links

1. [Currency Conversion Microservice](https://github.com/MalingaBandara/Currency-Conversion)
2. [Currency Exchange Microservice](https://github.com/MalingaBandara/Currency-Exchange)
3. [Naming Server Microservice](https://github.com/MalingaBandara/Naming-Server)
4. [API Gateway Microservice](https://github.com/MalingaBandara/API-Gateway)

## Learnings

- Understanding of microservices architecture and its benefits.
- Implementation of service discovery using Eureka.
- Experience with containerization and deployment using Docker.
- Utilization of distributed tracing for monitoring and debugging.
- Knowledge of message brokers for inter-service communication.
- Implementation of resilience patterns for fault tolerance.

## Project Structure and Code Explanation

1. **Microservice Configuration**:
   - Each microservice is configured with Spring Boot and registered with Eureka for dynamic discovery.
   - Example YAML configuration:
   ```yaml
   spring:
     cloud:
       gateway:
         routes:
           - id: currency-conversion
             uri: lb://CURRENCY-CONVERSION
             predicates:
               - Path=/currency-conversion/**
   ```

2. **Service Communication**:
   - Microservices communicate asynchronously using RabbitMQ, enhancing resilience and decoupling.
   - Zipkin provides distributed tracing for monitoring and debugging communication between services.

3. **Docker Compose Setup**:
   - The entire setup is containerized using Docker Compose for seamless orchestration.
   ```yaml
   version: '3.7'
   services:
     currency-exchange:
       image: currency-exchange-service
       ports:
         - "8000:8000"
       networks:
         - currency-network
       depends_on:
         - naming-server
   ```

## Commits and Project Evolution

1. **Initial Setup**: Created the project structure and configured Spring Boot dependencies.
2. **Microservices Implementation**: Developed currency conversion and exchange microservices.
3. **Service Discovery and Gateway**: Integrated Eureka and Spring Cloud Gateway.
4. **Docker and Asynchronous Communication**: Added Docker support and RabbitMQ for messaging.


## Learnings and Future Enhancements

This project helped deepen my understanding of microservices architecture, service discovery, containerization, and resilience patterns. Future enhancements may include adding OAuth2 security, implementing circuit breakers with Resilience4j, and incorporating advanced monitoring tools.

## Docker Compose Configuration

```yaml
version: '3.7' #docker compose version

services:
    currency-exchange:  # docker currency exchange
        image: bitlord00/mv2-currency-exchange-service:0.0.1-SNAPSHOT #image
        mem_limit: 700m  # memory limit for each container
        ports: 
            - "8000:8000"    # expose port 8000 to the host machine and map it to port 8000 in the docker container
        networks:
            - currency-network #network to connect the service to on the host machine and other services in docker-compose fileformat 
        depends_on:
            - naming-server  # dependency to make sure that the naming server is ready before starting this service
            - rabbitmq       # dependency to make sure RabbitMQ is ready before starting the service
        environment:
            EUREKA.CLIENT.SERVICEURL.DEFAULTZONE: http://naming-server:8761/eureka  # default eureka server address
            SPRING.ZIPKIN.BASEURL: http://zipkin-server:9411/  # default Zipkin server address
            # RabbitMQ Propertiesz
            RABBIT_URI: amqp://guest:guest@rabbitmq:5672 # default RabbitMQ connection URL (With Username, Password and Port)
            SPRING_RABBITMQ_HOST: rabbitmq               # default RabbitMQ Server hostname
            SPRING_ZIPKIN_SENDER_TYPE: rabbit            # default zipkin sender type is rabbitmq, can be set to "http" or "kafka" 



    currency-conversion:  # docker currency conversion service
        image: bitlord00/mv2-currency-conversion-service:0.0.1-SNAPSHOT #image
        mem_limit: 700m  # memory limit for each container
        ports: 
            - "8100:8100"   # expose port 8100 to the host machine and map it to port 8100 in the docker container
        networks:
            - currency-network #network to connect the service to on the host machine and other services in docker-compose fileformat 
        depends_on:
            - naming-server  # dependency to make sure that the naming server is ready before starting this service
            - rabbitmq       # dependency to make sure RabbitMQ is ready before starting the service
        environment:
            EUREKA.CLIENT.SERVICEURL.DEFAULTZONE: http://naming-server:8761/eureka  # default eureka server address
            SPRING.ZIPKIN.BASEURL: http://zipkin-server:9411/  # default Zipkin server address



    api-gateway:  # API gateway server
        image: bitlord00/mv2-api-gateway:0.0.1-SNAPSHOT #image
        mem_limit: 700m  # memory limit for each container
        ports: 
            - "8765:8765"   # expose port 8765 to the host machine and map it to port 8765 in the docker container
        networks:
            - currency-network #network to connect the service to on the host machine and other services in docker-compose fileformat 
        depends_on:
            - naming-server  # dependency to make sure that the naming server is ready before starting this service
            - rabbitmq       # dependency to make sure RabbitMQ is ready before starting the service
        environment:
            EUREKA.CLIENT.SERVICEURL.DEFAULTZONE: http://naming-server:8761/eureka  # default eureka server address
            SPRING.ZIPKIN.BASEURL: http://zipkin-server:9411/  # default Zipkin server address



    naming-server: # docker naming server
        image: bitlord00/mv2-naming-server:0.0.1-SNAPSHOT #image
        mem_limit: 700m  # memory limit for each container
        ports: 
            - "8761:8761"    # expose port 8761 to the host machine and map it to port 8761 of the container
        networks:
            - currency-network #network to connect the service to on the host machine and other services in docker-compose fileformat 



# docker run -p 9411:9411  openzipkin/zipkin:2.23

    zipkin-server: # docker zipkin server 
        image: openzipkin/zipkin:2.23 #image
        mem_limit: 300m  # memory limit for each container
        ports: 
            - "9411:9411"    # expose port 9411 to the host machine and map it to port 9411 of the container
        networks:
            - currency-network #network to connect the service to on the host machine and other services in docker-compose fileformat
        depends_on:
            - rabbitmq       # dependency to make sure RabbitMQ is ready before starting the service 
        restart: always #Restart if there is a problem starting up
        environment:
        # RabbitMQ Properties
            RABBIT_URI: amqp://guest:guest@rabbitmq:5672 # default RabbitMQ connection URL (With Username, Password and Port)


    rabbitmq:   # RabbitMQ Server
        image: rabbitmq:3.8.12-management #image
        mem_limit: 300m  # memory limit for each container
        ports: 
            - "5672:5672"      # expose port 5672 to the host machine and map it to port 5672 of the container
            - "15672:15672"    # expose port 15672 to the host machine and map it to port 15672 of the container
        networks:
            - currency-network #network to connect the service to on the host machine and other services in docker-compose fileformat 



Create a network called "currency-network" that containers can communicate with each other on
networks:
    currency-network:
```


## Running the Application

1. **Clone this repository:**

```bash
git clone https://github.com/MalingaBandara/Currency-Conversion-Exchange-Microservices

```

2. **Navigate to each microservice directory and build Docker images:**
```bash
cd currency-conversion-microservice
docker build -t currency-conversion .
```

3. **Run Docker containers for each microservice:**
```bash
docker run -d -p 8100:8100 currency-conversion
```

3. **Access the APIs:**

#### Currency Exchange Service:
  - http://localhost:8000/currency-exchange/from/USD/to/INR

#### Currency Conversion Service:
 - http://localhost:8100/currency-conversion/from/USD/to/INR/quantity/10
 - http://localhost:8100/currency-conversion-feign/from/USD/to/INR/quantity/10
 - Eureka (Naming Server): http://localhost:8761/

#### API-Gateway:
- http://localhost:8765/CURRENCY-EXCHANGE/currency-exchange/from/USD/to/INR
- http://localhost:8765/CURRENCY-CONVERSION/currency-conversion/from/USD/to/INR/quantity/10
- http://localhost:8765/CURRENCY-CONVERSION/currency-conversion-feign/from/USD/to/INR/quantity/10
- http://localhost:8765/currency-exchange/currency-exchange/from/USD/to/INR
- http://localhost:8765/currency-conversion/currency-conversion/from/USD/to/INR/quantity/10
- http://localhost:8765/currency-conversion/currency-conversion-feign/from/USD/to/INR/quantity/10

#### Currency Exchange Route (With Load balance - Eureka):
- http://localhost:8765/currency-exchange/from/USD/to/INR

#### Currency Conversion Route (With Load balance - Eureka):
- http://localhost:8765/currency-conversion/from/USD/to/INR/quantity/10
- http://localhost:8765/currency-conversion-feign/from/USD/to/INR/quantity/10
- http://localhost:8765/currency-conversion-new/from/USD/to/INR/quantity/10


## Author

- [Malinga Bandara](https://github.com/MalingaBandara)


## License

This project is licensed under the MIT License.
