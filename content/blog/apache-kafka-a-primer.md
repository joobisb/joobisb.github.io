+++
title = 'Apache Kafka: A Primer'
date = 2023-09-18T00:23:24+05:30
readTime = true
featuredImage = '/images/b17be644-da38-4e7a-b211-5f828dc7c59b.png'

+++

Nowadays it's very common to build systems focused on events where events drive the flow of data and interactions, aka *Event-driven architecture*.

Simply put, Apache Kafka is an event-streaming platform. It does the following,

* Single platform to publish and subscribe to events.
    
* Process real-time streams of events.
    
* All events are stored.
    

![](/images/b17be644-da38-4e7a-b211-5f828dc7c59b.png)

On a high level, Kafka has **Producers** and **Consumers** as shown above.

* Producers - They push events (obviously).
    
* Consumers - They consume. The arrow towards Kafka from the consumer indicates it **polls** the messages from Kafka.
    

Now let's breakdown Kafka,

![](/images/41b52fb9-4173-46b6-a7f7-ec895ca1a3bf.png)

A Kafka cluster consists of multiple brokers, each broker is running a Kafka process and stores the events in the disk. A broker can be easily referred to with any legacy terms such as machines/servers or can be containers or VMs etc.

We can deploy and manage a Kafka cluster or use any managed services such as [Confluent Cloud](https://www.confluent.io/), [AWS MSK](https://aws.amazon.com/msk/) etc.

![](/images/cc724065-b879-4b16-9fe6-e7ef94ebafd5.png)

You might be wondering what's the new component shown at the bottom. It's Zookeeper.

Kafka users Zookeeper for,

* Cluster management
    
* Metadata handling.
    
* Failure detection/recovery, leader election.
    
* ACLs etc.
    

A new protocol based on an event-based variant of the [Raft](https://raft.github.io/) consensus protocol was introduced in [KIP-500](https://cwiki.apache.org/confluence/display/KAFKA/KIP-500%3A+Replace+ZooKeeper+with+a+Self-Managed+Metadata+Quorum) to remove Apache Kafka’s dependency on ZooKeeper for metadata management called Apache Kafka Raft (**KRaft**).

With the release of Apache Kafka 3.5, Zookeeper is now marked **deprecated**. It is expected to be removed in version 4.0.

![](/images/3ca392ad-cc75-477c-884f-397f33aef4a2.png)

A **Topic** is a collection of related events. Data is propagated from producers to consumers through these topics. There can be multiple producers writing to a topic or a single producer writing to multiple topics. Similar is the case with the consumers as well.

Currently, Kafka can have conceptually as many as hundreds of thousands of topics in a Kafka cluster. With **KRaft** mode it is said to be in millions \[[Source](https://developer.confluent.io/faq/apache-kafka/topics-in-kafka/#:~:text=While%20there%20is%20no%20set,number%20of%20partitions%20in%20each.)\].

![](/images/5a7ca072-ec1c-4640-82d1-4bf169904c8c.png)

A topic which is actually only a logical grouping consists of **partitions** and each of these partitions can live on a separate broker in the cluster. So storing, writing and processing of messages can be split among the brokers in the cluster as shown below.

Here there are 3 topics, A, B and C each having 2 partitions.

![](/images/b08a7a69-378e-41f0-9f94-77b254164154.png)

Each partition has a configurable number of replicas (typically 3) for fault tolerance. So the distribution can change as follows.

![](/images/38b1da06-d01f-420b-9607-9ff1e91d4863.png)

Here, I've taken the case for topic A and its two partitions 0 and 1 which are replicated across 3 brokers. Similarly, for topics B and C it can have their own replicas as well across the 3 brokers. One of them will be the leader(L) and the rest are the followers(R1, R2).

Producers write events to the leader partition, similarly, consumers usually consume from the leader except for some special cases([KIP-392](https://cwiki.apache.org/confluence/display/KAFKA/KIP-392%3A+Allow+consumers+to+fetch+from+closest+replica)).

As a rule of thumb, it is recommended for each broker to have up to 4,000 partitions and for each cluster to have up to 200,000 partitions \[[Source](https://www.confluent.io/blog/apache-kafka-supports-200k-partitions-per-cluster/)\].

![](/images/3e8a66ef-54b8-4036-be1f-e1ca72165def.png)

A partition consists of **offsets,** the first entry is written to offset 0 then 1, 2 and it continues. All written events in a partition are immutable and ordered though we can set a retention period for events written (default: 7 days).

![](/images/f27bb2ed-ed28-4436-926a-7776c5a16f01.png)

A record or an event consists of

* Headers - these are optional metadata
    
* Key - if given, ensures all events with the same key end up in the same partition
    
* Value - relevant event data
    
* Timestamp - creation or ingestion time
    

![](/images/1c9b4fa8-2632-4960-aecd-833cf90b08b4.png)

*Producers* and *Consumers* use different strategies to push and consume events from the partition.

*Let's take a look at some of the producer* **partitioning strategies**.

* *Round Robin*: If the message has no key it follows this, but this does not provide any ordering guarantee as the messages can be pushed to any partition.  
      
    For eg: Consider a scenario where messages from different producers are being sent on the same topic simultaneously. Due to the round-robin nature of partitioning, these messages may not be stored in the order they were produced, which can lead to variations in the message sequence when consumed.  
    
* *Default strategy* with key: It chooses the partition through a hash function, `hash(key) % no_of_partitions`. This guarantees order since a message with the same key lands in the same partition.  
      
    By carefully choosing message keys, we can ensure related data lands in the same partition, preserving order and simplifying processing.  
    
* *A* *custom partition strategy* can also be defined.
    

*Consumers* subscribe to topics and pull messages based on the offset. It can start reading from the beginning of a partition or from a custom offset. Once read successfully the consumer commits the offset to the cluster (this is stored in an internal topic `__consumer_offsets`) indicating that the event has been read.

We can associate a **Group ID** with every consumer. All consumers with the same Group ID belong to a **Consumer Group**. As shown in the diagram above we have 2 consumers A and B which belongs to a Consumer Group.

For Topic A and B, Out of 3 of its partitions,  
2 -&gt; assigned to consumer A.  
1 -&gt; assigned to consumer B.  
This can be the other way around as well i.e. 2 assigned to B and 1 to A.

If there was a consumer C, the assignment would have been like,

![](/images/2a39a6cc-b7c5-42a8-a10a-0418715740e3.png)

Out of 3 of its partitions,  
1 -&gt; assigned to consumer A.  
1 -&gt; assigned to consumer B.  
1 -&gt; assigned to consumer C.

Let's add one more, consumer D

![](/images/4c283760-3075-4ed1-96c4-3fa0e14e9124.png)

Here consumer D is idle since there are no more partitions available to be assigned to it.

*Let's take a look at some of the consumer* **partition assignment strategies**.

* *Round Robin*: All partitions of the subscription, regardless of topic, will be spread evenly across the available consumers.
    
* *Sticky Partition*: A variant of round-robin but it makes the best effort at sticking to the previous assignment during a rebalance (**Rebalance** is the process in the which assignment of partition changes if a consumer joins or leaves the group or if the number of partitions in a topic changes).
    

*Range assignment strategy*: This goes through each topic in the subscription and assigns each of the partitions to a consumer, starting at the first consumer. This is useful when joining events from more than one topic the events need to be read by the same consumer.

Another concept to discuss is, **Topic Compaction (Log Compaction)** which ensures that Kafka will always retain at least the last known value for each message key within the log of data for a single topic partition.

As you can see below after compaction only the latest values for each key are retained after compaction. If the use case is such that, previous messages are not needed and the state can be restored from the latest values of the keys log compaction can be enabled and this saves us a lot of storage as well.

![](/images/1bcfbeeb-872b-4ea2-ad27-ad3a495ad14f.png)

Compaction gives a more granular retention mechanism so that we are guaranteed to retain at least the last update for each primary key.

Finally, there are certain patterns and various use cases related to event processing/streaming in general. So Kafka has built a whole ecosystem around this which includes.

* Kafka Connect
    
* Kafka Streams
    
* ksqlDB
    
* Schema Registry (Kafka does not include a schema registry, but there are third-party implementations available).
    

We will discuss more on this in further blogs.