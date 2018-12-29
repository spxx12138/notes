网上大多数资料都是基于1.x、2.x的，有很多已经过时。总结一下遇到的5.x以后的新特性。

### 5.x新特性 ###
 
1. **consistency -> wait_for_active_shards：** 我们在发送任何一个增删改操作的时候，都可以带上一个consistency参数，指明盖章操作的写一致性规则。也就是主分片与副本分片之间的一致。
    > 1. one：要求我们这个写操作，只要有一个primary shard是active活跃可用的，就可以执行
    > 2. all：要求我们这个写操作，必须所有的primary shard和replica shard都是活跃的，才可以执行这个写操作
    > 3. quorum：默认的值，要求所有的shard中，必须是大部分的shard都是活跃的，可用的，才可以执行这个写操作。
    > quroum的计算公式为quroum = int( (primary + number_of_replicas) / 2 ) + 1。
    > 
    但是，consistency检查是在put之前做的。虽然检查的时候，shard满足consistency设置的值，但是真正从primary写到replica之前，仍会出现shard挂掉，但Update Api会返回succeed。因此，这个检查并不能保证replica成功写入，甚至这个primary是否能成功写入也未必能保证。所以在5.x以后，将该参数改成了 **wait_for_active_shards** 因为这个更能清楚表述，而没有歧义。并且参数的值不再是原来的one、all、quroum，而是只接受正整数，这样也更自由点。
    
    consistency / wait_for_active_shards设置的是每个主分片与它对应的副本分片之间的一致性，所以 即使你有5个主分片，但是只有一个节点，副本分片数为0（因为一个节点上不会有两个相同的分片），这个时候你把参数设置为2，但是还是不能成功写入。


### 6.x新特性 ###
1. **每个index只能有一个type，推荐type名为_doc，7.x版本将会正式移除type**