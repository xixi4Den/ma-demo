# Slot Machine system - Demo

This project demonstrates a Slot Machine system with two microservices: 

1. [Slot Service](https://github.com/xixi4Den/MA.SlotService): Handles the slot machine logic.
2. [Reward Service](https://github.com/xixi4Den/MA.RewardService): Manages player progress and rewards.

The services rely on:
 - **Redis** as data storage (each service has dedicated Redis instance)
 - **RabbitMQ** as message broker
 - **Zipkin** as distributed tracing system
 - **Prometheus** as monitoring system

---

## Prerequisites

Before running the project, make sure you have the following installed:

- [**Docker**](https://www.docker.com/get-started)
- **Docker Compose**: Typically comes with Docker installation, but ensure it's up to date.

---

## Run with local build
Follow these steps to build images locally and run the services:
1. Clone [Demo](https://github.com/xixi4Den/ma-demo) repository
2. Clone [Slot service](https://github.com/xixi4Den/MA.SlotService) repository
3. Clone [Reward service](https://github.com/xixi4Den/MA.RewardService) repository
4. Ensure all repositories are placed at the same level:
```
<root>
└───ma-demo
│   │   docker-compose.yml
│   │
│   └───config
│       │   missions-config.json
│
└───MA.SlotService 
│   
└───MA.RewardService
```
5. Modify `/ma-demo/config/missions-config.json` file (if requred)
6. Run `docker-compose up --build` to build images and start the services with their dependencies
7. Access services
  - Slot service
    - [Swagger UI](http://localhost:8081/swagger)
    - [Healh check](http://localhost:8081/health)
  - Reward service
    - [Swagger UI](http://localhost:8082/swagger)
    - [Healh check](http://localhost:8082/health)
  - RabbitMQ:
    - [management interface](http://localhost:15672) (login: `guest`, password: `guest`)
- [Zipkin](http://localhost:9411)
- [Prometheus](http://localhost:9090)
8. Run `docker-compose down` to remove services and their dependencies


## Run with images from DockerHub

Follow these steps to run the services from images published to DockerHub:
1. Clone [Demo](https://github.com/xixi4Den/ma-demo) repository
2. Modify `/ma-demo/config/missions-config.json` file (if requred)
3. Run `docker-compose -f docker-compose.images.yml up` to start the services and their dependencies
4. Access services
  - Slot service
    - [Swagger UI](http://localhost:8081/swagger)
    - [Healh check](http://localhost:8081/health)
  - Reward service
    - [Swagger UI](http://localhost:8082/swagger)
    - [Healh check](http://localhost:8082/health)
  - RabbitMQ:
    - [management interface](http://localhost:15672) (login: `guest`, password: `guest`)
- [Zipkin](http://localhost:9411)
- [Prometheus](http://localhost:9090)
5. Run `docker-compose -f docker-compose.images.yml down` to remove services and their dependencies

## Configuration

### Missions configuration

Missions are configured using a JSON file. In the current setup, the `/config` folder is mounted to `/opt/reward/config` in the Reward service container. You can modify the configuration using the `/config/missions-config.json` file.

Here is default configuration in `/config/misions-config.json` file
```
{
	"missions": [
		{
			"rewards":[
				{
					"name": "spins",
					"value": 10
				}
			],
			"pointsGoal": 10
		},
		{
			"rewards":[
				{
					"name": "coins",
					"value": 10
				}
			],
			"pointsGoal": 20
		},
		{
			"rewards":[
				{
					"name": "coins",
					"value": 100
				},
				{
					"name": "spins",
					"value": 100
				}
			],
			"pointsGoal": 10
		}
	],
	"repeatedIndex": 1
}
```

## Technical considerations

![Sequence diagram](/images/diagram1.png)

1. Services communicate asynchronously using RabbitMQ as a message broker, with the MassTransit library as an abstraction layer
2. RabbitMQ provides _at least once_ delivery guarantee. To meet the system's requirement for _exactly once_ delivery, I implemented a primitive deduplication mechanism at application layer
3. The Reward service uses Redis transactions as optimistic locks to ensure consistent mission progress updates. In scenarios where the service is scaled out, a race condition may occur. To mitigate this, the following strategies could be considered:
	- add retries if the conflict rate is low.
	- using RabbitMQ’s [Single Active Consumer](https://www.rabbitmq.com/docs/consumers#single-active-consumer) feature to ensure a single consumer processes messages per queue, though this limits scalability.
	- change optimistic lock to pessimistic lock for high conflict rates, which may impact throughput.
4. _CoinsRewardGrantedEvent_ is published by the Reward service but remains unconsumed in the current scope. In my view, the Coins entity logically belongs to a dedicated service, such as a Wallet service, responsible for managing virtual goods. Additional requirements would help refine this decision.
5. Given the asynchronous nature of communication, slot balance and coin balance are updated asynchronously, which may present challenges for the frontend. While frontend design was out of scope, potential solutions include:
	- polling or long polling
	- WebSocket (I’ve included a diagram illustrating a potential architecture) ![Web socker diagram](/images//diagram2.jpg)
6. Authentication was out of scope for this assignment. _UserId_ is passed as a plain integer in HTTP header.
7. Both services are covered by unit and integration tests. Integration tests run with Redis, managed by TestContainers.
8. Both services expose OpenTelemetry traces and metrics to ensure the system’s observability—a critical aspect of distributed systems.

