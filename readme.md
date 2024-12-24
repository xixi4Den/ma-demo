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
Follow these steps to run the services locally with Docker Compose:
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
6. Run `docker-compose up` to build and start the services and their dependencies
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
6. Run `docker-compose down` to remove services and their dependencies


## Run with built images

TODO

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
