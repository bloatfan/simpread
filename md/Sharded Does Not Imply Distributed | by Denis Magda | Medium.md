> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/@magda7817/sharded-does-not-imply-distributed-572fdafc4040)

> Sharding is a technique that distributes data and load across several standalone database instances. ......

[

![](https://raw.githubusercontent.com/lslz627/PicGo/master/1*oYE9KKE5RHSFPj6iwb8zJw.jpeg)

](https://medium.com/@magda7817?source=post_page-----572fdafc4040--------------------------------)

Sharding is a technique that distributes data and load across several standalone database instances. This method leverages horizontal scalability by splitting the original dataset into shards, which are then distributed across multiple database instances.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/1*yg3PV8O2RO4YegyiYeiItA.png)

But, even though the verb “distributes” appears in the definition of sharding, a sharded database is not a distributed one.

Every sharding solution has one critical component in its architecture. This component can go by various names, including coordinator, router, or director:

![](https://raw.githubusercontent.com/lslz627/PicGo/master/1*kp39_8mQ0E9bIO0Lw3PGFw.png)

The coordinator is the sole component aware of data distribution. It maps client requests to specific shards and then to the corresponding database instance. This is why clients must always route their requests through the coordinator.

For example, if a client wants to insert a new record into the `Car`table, the request first goes to the coordinator. The coordinator maps the record’s primary key to one of the shards and then forwards the request to the database instance responsible for that shard.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/1*YNUB6y8WJnp0CCVAXSjQ0g.png)

In the schema above, first, the coordinator maps key `121` to shard `10` and, second, inserts the record into table `car_10` that is stored on the database instance owning shard `10`

However, one question remains: Why is the coordinator even needed in sharding solutions? The answer is straightforward. The shards are stored on database instances designed for single-server deployments.

_These database instances do not communicate with each other, nor do they support any protocols that would facilitate such communication. Unaware of each other, they exist in their own isolated environments, oblivious to the fact that they are part of a larger system._

Consequently, the coordinator is indispensable in sharding solutions. If you’re interested in delving deeper into sharded database architectures, consider exploring CitusData or Azure CosmosDB for PostgreSQL, Vitess for MySQL, Oracle Distributed Autonomous Database, and MongoDB Sharded Cluster.

Much like sharded database solutions, distributed databases also employ similar sharding techniques to distribute data and load across a cluster of database nodes. However, unlike sharding solutions, distributed databases do not rely on a coordinator component.

**Distributed databases are built on a shared-nothing architecture**, which doesn’t have a single component, like the coordinator, burdened with making numerous decisions:

![](https://raw.githubusercontent.com/lslz627/PicGo/master/1*deOgcXccWs9lKUSgLPNOww.png)

_All nodes in the cluster are aware of each other and, consequently, the data distribution. By communicating directly, each node can route a client request to the appropriate shard owner. Additionally, they can execute and coordinate multi-node transactions. When scaling to more nodes, the cluster automatically rebalances and splits shards. The nodes maintain redundant copies of data (based on a configured replication factor) and can continue operations without downtime, even if some nodes fail._

All of this operates transparently for the client, who simply needs to establish a connection with any of the nodes and allow that node to manage the distributed aspects.

For example, a client might connect to `node1` and insert a new `Car` record with the id `121`. If `node1` is the owner of the record’s shard, then it will store the record locally and employ a consensus algorithm to replicate the change to a subset of other nodes. If not, `node1` will forward the record to the shard’s owner, which might be `node4`.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/1*weEdq2BxIpf6GiLjipns5Q.png)

If you’re interested in exploring the architectures of genuine distributed databases, consider looking into Google Spanner, YugabyteDB, CockroachDB, Apache Cassandra, or Apache Ignite.

In the realm of databases, sharding and distribution are often conflated, but they serve distinct purposes.

While sharding involves splitting data across multiple standalone instances, it doesn’t inherently mean the system is distributed. The presence of a coordinator in sharding solutions, which directs client requests to the appropriate shard, underscores this distinction.

On the other hand, distributed databases, built on a shared-nothing architecture, lack this centralized coordinator. Nodes in these systems are aware of each other, manage data distribution, and handle client requests seamlessly.

Both architectures have their merits, and understanding their nuances is crucial for informed database design and selection.