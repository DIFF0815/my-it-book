问题1:懂Redis事务么？
`正常版`：Redis事务是一些列redis命令的集合,blabla...
`高调版`: 我们在生产上采用的是Redis Cluster集群架构，不同的key是有可能分配在不同的Redis节点上的，在这种情况下Redis的事务机制是不生效的。其次，Redis事务不支持回滚操作，简直是鸡肋！所以基本不用！

问题2:Redis的多数据库机制，了解多少？
`正常版`：Redis支持多个数据库，并且每个数据库的数据是隔离的不能共享，单机下的redis可以支持16个数据库（db0 ~ db15）
`高调版`: 在Redis Cluster集群架构下只有一个数据库空间，即db0。因此，我们没有使用Redis的多数据库功能！

问题3:Redis集群机制中，你觉得有什么不足的地方吗？
`正常版`: 不知道
`高调版`: 假设我有一个key，对应的value是Hash类型的。如果Hash对象非常大，是不支持映射到不同节点的！只能映射到集群中的一个节点上！还有就是做批量操作比较麻烦！

问题4:懂Redis的批量操作么？
`正常版`: 懂一点。比如mset、mget操作等，blabla
`高调版`: 我们在生产上采用的是Redis Cluster集群架构，不同的key会划分到不同的slot中，因此直接使用mset或者mget等操作是行不通的。

问题5:那在Redis集群模式下，如何进行批量操作？
`正常版`:不知道
`高调版`:这个问题其实可以写一篇文章了，改天写。这里说一种有一个很简单的答法，足够面试用。即:
如果执行的key数量比较少，就不用mget了，就用串行get操作。如果真的需要执行的key很多，就使用Hashtag保证这些key映射到同一台redis节点上。简单来说语法如下

> **对于key为{foo}.student1、{foo}.student2，{foo}student3，这类key一定是在同一个redis节点上。因为key中“{}”之间的字符串就是当前key的hash tags， 只有key中{ }中的部分才被用来做hash，因此计算出来的redis节点一定是同一个!**

`ps`:如果你用的是Proxy分片集群架构，例如Codis这种，会将mget/mset的多个key拆分成多个命令发往不同得redis实例，这里不多说。我推荐答的还是redis cluster。

问题6:你们有对Redis做读写分离么？
`正常版`:没有做，至于原因额。。。额。。。额。。没办法了，硬着头皮扯~
`高调版`:不做读写分离。我们用的是Redis Cluster的架构，是属于分片集群的架构。而redis本身在内存上操作，不会涉及IO吞吐，即使读写分离也不会提升太多性能，Redis在生产上的主要问题是考虑容量，单机最多10-20G，key太多降低redis性能.因此采用分片集群结构，已经能保证了我们的性能。其次，用上了读写分离后，还要考虑主从一致性，主从延迟等问题，徒增业务复杂度。