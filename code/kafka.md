# Kafka
## Prerequisite
- Knowledge
  - Node.JS 
  - Experience with designing distributed systems
- Tools
  - Node.js: [Download Node.JS](https://nodejs.org/en)
  - Docker: [Download Docker](https://www.docker.com)
  

## Commands
- Start Zookeper Container and expose PORT `2181`.
```bash
docker run -p 2181:2181 zookeeper
```
- Start Kafka Container, expose PORT `9092` and setup ENV variables.
```bash
docker run -p 9092:9092 `
-e KAFKA_ZOOKEEPER_CONNECT=192.168.0.101:2181 `
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.0.101:9092 `
-e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 `
confluentinc/cp-kafka:7.4.0
```

## Running Locally

- node admin.js
- Run Multiple Consumers
```bash
node consumer.js <GROUP_NAME>
```
- Create Producer
```bash
node producer.js
```
```bash
> alex south
> alex north
```