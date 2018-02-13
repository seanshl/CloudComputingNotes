# What does Apache Kafka do
1. Decouple data streams
2. Target system subscribe from kafka.
3. Distributed, resilient architecture, fault tolerant
4. Horizontal scalability
5. High performance
6. Created by Linkedin(then open-source)
7. Usage
	1. Messaging System
	2. Activity Tracking
	3. Gather metrics from many different locations
	4. Application Log gathering
	5. Stream processing
	6. Decoupling of system dependencies
	7. Integration with Spark, Flink, Storm, Hadoop...
	
# Architecture
1. Source Systems
2. Producers
3. Kafka  -> ZooKeeper
4. Consumer
5. Target Systems

# Enterprise Architecture
1. Data Producers
	1. Apps
	2. Website
	3. FInacial Systems.
	4. Databases...
2. Realtime: Spark/Storm/ --> REaltime analytics, Dashboards/Alerts Apps/Consumer
3. Batch

# Conceptions
1. Topics
	* a particular stream of data 
	* Similar to table in DB (no constraints)
	* You can have as many as you want 
	* identified by name
2. Partitions
	* Topics are split in partitions
	* Partition is part of topic
	* each partion is ordered
	* Each message within a partition gets an incremental id, called offset.
	* Offsets only have a meaning for a specific partition
	* Order only guarantee in one parition
	* Data only kept for a limited time (default 2 weeks)
	* Data immutable after writen in parition
	* You push data to topic
	* Data is assigned randomly to a partition unless a key is provided
	* More partition = more parallelism
3. Brokers
	* Kafka cluster is composed of multiple brokers(servers)
	* identified by id
	* once connect to any broker, you connect to the entire cluster
	* Not every broker has every topic
4. Topic replication factor
	* replication factor > 1 (2 or 3)
	* at any time only one broker can be a leader for a given partition
	* only the leader can receive and serve data for a partition
	* other brokers will synchronize the data
	* one leader, multiple ISR(in-sync replica)
5. Producers
	* Write data to topics
	* only have to specify the topic name and one broker to connect to 
	* If a key is sent, then the producer has the guarantee that all messages for that key will always go to the same partition
	* this enables to guarantee ordering for a pecific key
6. Consumers
	* read data from a topic
	* only have to specify the topic name and one broker to connect to .
	* Data is read in order for each partitions.
	* Messages are read in order
	* Partitions are read parally
7 Consumer Groups
	* cannot be more than partitions
	
