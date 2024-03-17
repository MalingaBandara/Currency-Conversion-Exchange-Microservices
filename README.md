# Currency Conversion and Exchange API with Microservices

This project demonstrates the development of a currency conversion and exchange API using Java Spring Boot Cloud with microservices architecture deployed with Docker. The project is split into four microservices:

1. **Currency Conversion Microservice**: Responsible for converting currency.
2. **Currency Exchange Microservice**: Handles currency exchange operations.
3. **Naming Server Microservice**: Provides service discovery using Eureka.
4. **API Gateway Microservice**: Serves as the entry point for accessing the API.

## Technologies Used

- **Java Spring Boot Cloud**: Framework for building microservices.
- **Docker**: Used for containerization and deployment.
- **Eureka**: Service discovery tool.
- **Zipkin**: Distributed tracing system.
- **H2 Database**: In-memory database for development.
- **RabbitMQ**: Message broker for communication between microservices.
- **Resilience4j**: Library for implementing resilience patterns.

## Project Structure

- `currency-conversion-microservice`: Source code for the currency conversion microservice.
- `currency-exchange-microservice`: Source code for the currency exchange microservice.
- `naming-server-microservice`: Source code for the naming server microservice.
- `api-gateway-microservice`: Source code for the API gateway microservice.


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
git clone <main_repository_url>

```

2. **Navigate to each microservice directory and build Docker images:**
```bash
cd currency-conversion-microservice
docker build -t currency-conversion .
```

3. **Run Docker containers for each microservice:**
```bash
docker run -d -p 8080:8080 currency-conversion
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

## Learnings

- Understanding of microservices architecture and its benefits.
- Implementation of service discovery using Eureka.
- Experience with containerization and deployment using Docker.
- Utilization of distributed tracing for monitoring and debugging.
- Knowledge of message brokers for inter-service communication.
- Implementation of resilience patterns for fault tolerance.

## Repository Links

1. [Currency Conversion Microservice](https://github.com/MalingaBandara/Currency-Conversion)
2. [Currency Exchange Microservice](https://github.com/MalingaBandara/Currency-Exchange)
3. [Naming Server Microservice](https://github.com/MalingaBandara/Naming-Server)
4. [API Gateway Microservice](https://github.com/MalingaBandara/API-Gateway)

## Author

- [Malinga Bandara](https://github.com/MalingaBandara)

