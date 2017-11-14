
## Cassandra

### References
* [Cassandra架构理解](http://zqhxuyuan.github.io/2015/08/25/2015-08-25-Cassandra-Architecture/)
* [一致性哈希算法详解](http://blog.csdn.net/cywosp/article/details/23397179/)
### Distributed Hashtable (DHT)

### 架构与通信
1. **Non-Centralized Database**
	* 不同于以往的主从分布式的中心化结构, 采用环形结构结构，无单点，无中心的p2p结构. 
	* Cassandra集群中的节点**没有主次之分**, 数据分布于集群中各节点，各节点**每秒通过gossip协议**通信

2. **节点间通信**
	* 通信协议为gossip, 通过gossip在集群中的节点间交换位置和状态信息, 通过Gossip协议，它们可以知道集群中有哪些节点，以及这些节点的状态如何
	* gossip每秒钟运行一次，与至多3个其他节点交换信息，这样所有节点可以迅速了解到集群中其他节点的信息
	* 每一条Gossip消息上都有一个版本号，节点可以对接收到的消息进行版本比对，从而得知哪些消息是我需要更新的，哪些消息是我有而别人没有的，然后互相倾诉吐槽，确保二者得到的信息相同.
	* Cassandra启动时，会启动Gossip服务，Gossip服务启动后会启动一个任务GossipTask，这个任务会周期性地与其他节点进行通信。GossipTask是位于org.apache.cassandra.gms.Gossip类下的一个内部类

3. **失败检测与恢复**
	* 通过gossip来检测其他节点是否正常以避免将请求路由至不可达或者性能差的节点
	* 对于失败节点，其他节点会通过gossip定期与之联系来查看是否恢复
	* 一旦某节点被标记为失败，其错过的写操作会有其他replicas存储一段时间. (需开启hinted handoff，若节点失败的时间超过了max_hint_window_in_ms，错过的写不再被存储). Down掉的节点经过一段时间恢复后需执行repair操作，一般在所有节点运行nodetool repair以确保数据一致。
	
4. **核心组件**
	1. gossip
		* 点对点之间的通信方式
	2. partitioner
		* 决定如何在集群中的节点间分发数据，即在哪个节点放置数据的**第一个**replica.
	3. replica placement strategy
		* 复制策略，确定哪个节点放置复制数据， 以及复制的份数. 
		* Cassandra在集群中的偶个节点存储数据的多个备份来确保可靠性和容错性
	4. snitch
		* 定义一个网络拓扑图
		* 用来确定如何放置复制数据
		* 高效地路由
		* 定义复制策略用来放置replicas和路由请求所使用的路由信息
	5. cassandra.yaml
	
### Under the hood of Partitioner: DHT（一致性哈希）
1. **解决问题**
	* 一般的哈希算法 hash_value = hash_code % n.
	* 动态增删节点使n发生变化导致重新计算哈希，导致数据迁移量太大
2. **结构**
	* 环状结构，环可视作hash空间，也就是hash的值空间，范围为0 - 2^32
3. **原理**
	1. 添加节点:
		* 将节点的某个特征值,例如ip地址进行哈希获得hash code, hashcode进行某种散列方式映射到hash值空间上.
		* 从该节点逆时针向前找到上一个节点，将两节点之间的数据迁移到该节点上
			* 从顺时针的下一个节点找到数据
	2. 删除节点：
		* 顺时针将失效节点的数据迁移到顺时针的下一个节点上去.
	2. 查询节点: 将所需要存储的对象的进行哈希，获得它在环上的位置，然后顺时针找到最近的节点机器，将其存放上去
	4. 引入虚拟节点，就是实际节点在hash空间中的复制品. 来更均匀地分布节点.
4. **Partitioner**
	* 就是DHT中的哈希函数
	* 对存储对象的primary key进行hash
	
### 数据复制策略
1. SimpleStrategy:
	* 第一个replica放在partitioner确定的节点中
	* 其余的放在该节点顺时针方向的后续节点中
2. NetworkTopologyStrategy:	
	* 适用于较复杂的多数据中心
	* 可以指定在每个数据中心分别存储多少分replica

### 客户端连接


### Misc
1. 如何区分是不是distributed database?
	* 不以防止单点失败为目的的多数据库

### Write
1. Cassandra拥有非常高的写性能
2. 存储结构类似LSM树，存储引擎以追加的方式顺序写入磁盘连续存储数据，写入是可以并发写入的，不像B+树需要加锁，写入速度很快。levelDB和Hbase都是类似的存储结构。
2. 过程：
	
