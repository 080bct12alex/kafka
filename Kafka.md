# Introduction to Apache Kafka
- Kafka is a  **distributed, high-throughput, fault-tolerant event streaming platform**. 


#### *The Problem Kafka Solves: Database Throughput Limitations*

- Real-time applications like Zomato's food delivery tracking, Uber/Ola ride updates, or Discord's chat system generate massive amounts of data continuously. 

- Traditional databases (e.g., SQL) struggle with this high volume of concurrent write operations due to their low throughput (operations per second measures the rate of processing reads and writes). 

- Attempting frequent inserts (e.g., every second from thousands of drivers) can lead to database slowdowns, crashes, and significant delays , making the system non-real-time. 

- For example, Uber/Ola's real-time driver data needs to be consumed by multiple services (fair calculation, analytics, customer service), each potentially performing operations on separate database tables, which would overwhelm a traditional database. 

- Kafka addresses this by providing high throughput, enabling it to handle millions of records per second without performance degradation. 


*Kafka vs. Databases: A Complementary Relationship*

- Kafka is not an alternative to traditional databases; while it offers high throughput for processing large amounts of data quickly but  its storage capacity is limited to temporary data and cannot retain it long-term. 

- In contrast, databases may have lower throughput but can store billions of records permanently, allowing for complex queries and data persistence. 

- Ultimately, Kafka excels in data throughput but lacks the storage capability and querying flexibility that databases provide.

- So kafka solve the problem of throughput and database solve the problem of storage 

- Thus used kafka  and database as a unit  

- *Key point*:

  - Kafka is not an alternative to databases; they are complementary tools.

---


Kafka's  Architecture
---

The architecture of Kafka is designed to handle large data volumes produced by multiple sources / Producers , such as ride-sharing services like OLA and Uber. 

**Kafka Components**

![Kafka Core Components](https://i.ibb.co/mFT06mX2/Kafka-Core-Components.png)

In Kafka, key components include producers and consumers, where producers generate data and consumers consume messages. 


Producers  generate vast amounts of raw, unprocessed data. 


This data is fed into Kafka, which leverages its high throughput to ingest and store it temporarily. 

Instead of directly posting data to a database, which would crash the system, Kafka serves as a buffer that allows for efficient data handling and processing, ultimately merged it into a single database transaction.

Consumers (e.g., various backend services like analytics, fair calculation, customer-facing apps) subscribe to Kafka, consume the data, process it, and then perform bulk inserts into their respective databases. 

This approach significantly reduces the load on databases by converting many small, frequent writes into fewer, larger transactions. 

This setup not only enhances throughputs but also provides flexibility for multiple consumers to process the data simultaneously, thereby optimizing system performance.

Here is a flowchart illustrating Kafka's role in a real-time system:

![Kafka in Real-Time Systems](https://i.ibb.co/6RWxLpNf/Kafka-in-Real-Time-Systems.png)


---

**Kafka Internals: Topics and Partitions**

A Kafka server contains *Topics, which are logical categories / partition for organizing messages, similar to chat groups (e.g., "Rider Updates", "Hotel Updates")*  allowing users to categorize and manage data efficiently. 

Producers publish messages to specific topics.

Different topics can be created for various purposes, such as rider updates or hotel information, which helps in organizing the data flow. 


*Topics are further divided into Partitions to distribute data and handle high message volumes, akind to database sharding.* 

Data is partitioned based on a consistent logic (e.g., geographical location like North/South India, user ID), not time. 

Partitions are indexed starting from 0 (e.g., 0, 1, 2, 3). 

Producers use custom logic to route messages to specific partitions based on criteria (e.g., if (location == "North") use partition 0). 
Each partition has its own database. 


*Consumer Interaction with Partitions*

Key point: Self Balancing among individual consumer

By default, a single consumer subscribes to and consumes messages from all partitions of a topic . 

If multiple consumers are added to a topic, Kafka automatically balances the load, assigning partitions among them. 

If the number of consumers exceeds the number of partitions, the excess consumers will remain idle. 

Key Rule: While one consumer can consume multiple partitions, a single partition can only be consumed by one consumer at a time . 



*Consumer Groups in Kafka :* 

- Enabling Flexible Consumption  using queue and pub / sub

Consumers in Kafka are always part of a Consumer Group. 

Partition assignment and balancing occur at the group level. 

If a consumer group has only one consumer, that consumer receives messages from all partitions. 

If more consumers join the same group, partitions are balanced among them. 

If the number of consumers  in the group exceeds the number of partitions, the excess consumers will remain idle. 

If there are multiple consumer groups, each group independently receives all messages from all partitions. 

key point : Self balancing in group level  i.e Even if the consumers from one group consume all partitions,there is system of consuming that partition  by consumer of other groups.


*Kafka's Dual Role: Queue and Pub/Sub*

Kafka can be configured to act as both a Queue (First-In, First-Out) and a Publish-Subscribe (Pub/Sub) system by leveraging consumer groups.

---
*Kafka Infrastructure and Development Setup*
-
Prerequisites:  Node.js , Docker, and experience with designing distributed systems. 

**ZooKeeper**

An open-source tool internally used by Kafka for managing system state, replicas, and consumer balancing. 

It runs as a separate service, typically on port 2181

Tool to manage topic partitions and consumer interactions.

**Kafka Broker** 

The Kafka server itself, usually runs on port 9092

It needs configuration to connect to ZooKeeper and to advertise its network listeners (IP address and port). 

**kafkajs Library**

A Node.js library used to interact with Kafka. 

---
**Roles in Kafka Interaction**

*Admin*

Responsible for setting up the Kafka infrastructure, including creating topics and defining their partitions. 

*Producer*

Sends messages to specific Kafka topics and partitions.

*Consumer* 

Receives and processes messages from Kafka topics, operating within a consumer group. 

---
**Setting Up Kafka Environment**

The setup process for a Kafka environment involves using the ZooKeeper tool to manage topic partitions and consumer interactions.

Essential steps include configuring Docker to run ZooKeeper and Kafka services, specifying port settings, and defining environmental variables for connectivity. 


**Admin Functions in Kafka**

The infrastructure setup in Kafka involves creating topics, partitions, and using producers and consumers for message handling. 

The process starts with installing necessary libraries, 
followed by creating an admin instance to connect to a broker, which is configured at port 9092. 

Topics are created with specified parameters, such as the number of partitions, to handle updates, allowing for efficient data management in a Kafka environment.



*Kafka Producer Setup*

The first step involves importing the Client module, which is required for all Kafka components. 

A Producer instance is then created using kafka.Producer. 

Before sending messages, the producer must connect to the Kafka cluster, which is done using await producer.connect(). 

*Sending Messages*

Messages are sent using the producer.send() method. 

When sending a message, the topic to which the message should be sent must be specified. 

Multiple messages can be sent through the producer. 

Here is a flowchart illustrating the Kafka producer process:


![Kafka Producer Process](https://i.ibb.co/nsfRRxZT/Kafka-Producer-Process.png)


*Message Structure*

Each message consists of a key and a value. 

The key can be an identifier, such as a rider's name (e.g., "Tony Stark"). 
The value typically contains the actual data, often formatted as a JSON stringified object (e.g., {"riderName": "Tony Stark", "location": "Hyderabad"}). 
Messages can also be explicitly assigned to a specific partition within the topic, for example, partition 0. 

After producing messages, the process concludes. 