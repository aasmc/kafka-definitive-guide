# Configuring the Broker

Most of the options can be left at the default settings, though, as they deal with tuning 
aspects of the Kafka broker that will not be applicable until you have a specific use 
case that requires adjusting these settings.

## General Broker Parameters

### broker.id

Every Kafka Broker must have a unique (within a Kafka cluster) id. By default it is 0.

### listeners

A comma-separated list of URIs that we listen on with the listener names. If the listener
name is not a common security protocol, then another config listener.security.protocol.map
must also be configured. A listener is defined as <protocol>://<hostname>:<port>. An example 
of a legal listener config is PLAIN TEXT://localhost:9092,SSL://:9091

### zookeeper.connect

The location of the ZooKeeper used for storing the broker metadata.
The format for this parameter is a semicolon-separated list of hostname:port/path strings.
/path is an optional ZooKeeper path to use as a chroot environment for the Kafka cluster.
If it is omitted, the root path is used.

### log.dirs

Directory to store log segments (Kafka persists all messages to disk) and these log segments
are stored in the directory specified in the log.dir configuration. For multiple directories
the config log.dirs is preferable. It is a comma-separated list of paths on the local system.
If more than one path is specified, the broker will store partitions on them in a “least-used”
fashion, with one partition’s log segments stored within the same path. Note that the broker
will place a new partition in the path that has the least number of partitions currently 
stored in it, not the least amount of disk space used, so an even distribution of data across 
multiple directories is not guaranteed.

### num.recovery.threads.per.data.dir

Kafka uses a configurable pool of threads for handling log segments. Currently, this thread pool 
is used:
- When starting normally, to open each partition’s log segments
- When starting after a failure, to check and truncate each partition’s log segments
- When shutting down, to cleanly close log segments

By default, only one thread per log directory is used. As these threads are only used during
startup and shutdown, it is reasonable to set a larger number of threads in order to parallelize
operations.

When setting this parameter, remember that the number configured is per log directory specified 
with log.dirs. This means that if num.recovery.threads.per.data.dir is set to 8, and there 
are 3 paths specified in log.dirs, this is a total of 24 threads.

### auto.create.topics.enable

By default Kafka creates topics when:
- a producer starts writing messages to the topic
- a consumer starts reading messages from the topic
- any client requests metadata for the topic

To disable this (in case you are managing topic creation manually), set this config to false.

### auto.leader.rebalance.enable

In order to ensure a Kafka cluster doesn’t become unbalanced by having all topic leadership 
on one broker, this config can be specified to ensure leadership is balanced as much as 
possible. It enables a background thread that checks the distribution of partitions at 
regular intervals (this interval is configurable via 
***leader.imbalance.check.interval.seconds***). If leadership imbalance exceeds another config,
***leader.imbalance.per.broker.percentage***, then a rebalance of preferred leaders for 
partitions is started.

### delete.topic.enable

Enables or disables deletion of topics in the cluster.

