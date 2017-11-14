
## Cassandra

### References
* [Cassandra架构理解](http://zqhxuyuan.github.io/2015/08/25/2015-08-25-Cassandra-Architecture/)
* [一致性哈希算法详解](http://blog.csdn.net/cywosp/article/details/23397179/)
* [Cassandra原理介绍](http://blog.csdn.net/doc_sgl/article/details/51068131)
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
	* hinted handoff: 它的过程是如果负责某个key值的某个节点宕机了，另一个节点会被选择作为其临时切换点，以临时保存在故障节点上面的写操作。这些写操作被单独保存起来，直到故障节点恢复正常，临时节点会把这些写操作重新迁移给刚刚恢复的节点。Dynamo 论文中提到一种叫“sloppy quorum”的方法，它会把通过 Hinted Handoff 写成功的临时节点也计算在成功写入数中。但是Cassandra和Voldemort并不会将临时节点也算在写入成功节点数内，如果写入操作并没有成功写在W个正式节点中，它们会返回写入失败。当然，Hinted Handoff 策略在这些系统中也有使用，不过只是用在加速节点恢复上。
	
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
1. 协调者(Coordinator)
	* client连接至节点并发出read/write请求时，该节点充当client端应用与包含请求数据的节点(或replica)之间的协调者，它利用配置的partitioner和replicaplacement策略确定那个节点当获取请求。
2. 写请求
	* 协调者(coordinator)将write请求发送到拥有对应row的所有replica节点，只要节点可用便获取并执行写请求。
	* 写一致性级别(write consistency level)确定要有多少个replica节点必须返回成功的确认信息。成功意味着数据被正确写入了commit log和memtable。
	* ![](http://img.blog.csdn.net/20150830123130980)
	* 上例为单数据中心，12个节点，复制因子为3，写一致性等级为ONE的写情况(红色的箭头表示只要一个节点成功写入,便可立即返回给客户端)
3. 读请求
	1. 读请求区分为
		* Direct Read
		* Repair Read
	2. 过程:
		1. 根据一致性级别与被确定的所有replica联系，被联系的节点返回请求的数据
		2. 若多个节点被联系，则来自各replica的row会在内存中作比较，若不一致，则协调者使用含最新数据的replica向client返回结果。
		3. 协调者在后台联系和比较来自其余拥有对应row的replica的数据，若不一致，会向过时的replica发写请求用最新的数据进行更新:read repair。
	3. 数据一致性
		1. 协调者进行比较操作
		2. 比较操作只需要两个副本的是时间戳
		3. 会用md5判断值是否相同
		4. 在确定哪个是最新的后，那个副本才把查询结果传送给协调者，之前的时候不传查询结果
		5. 协调者将结果传送给客户端
	 

### Misc
1. 如何区分是不是distributed database?
	* 不以防止单点失败为目的的多数据库

### 存储结构
1. SSTable: Sorted Strings Table

### Read
* ![](http://img.blog.csdn.net/20150830142808484)
* 流程
	1. 检查boomfilter
		* 每个table有多个sstable, 每个sstable自带一个boomfilter,使用boomfilter来检查partition key是否在这个sstable中
		* 这一步在磁盘IO前结束
	2. 若存在pk, 则检查parition key cache
		

### Write
* Cassandra拥有非常高的写性能
* 存储结构类似LSM树，存储引擎以追加的方式顺序写入磁盘连续存储数据，写入是可以并发写入的，不像B+树需要加锁，写入速度很快。levelDB和Hbase都是类似的存储结构。
* ![](http://img.blog.csdn.net/20150830130032129)
* 流程
	1. 首先由Commit Log捕获写事件并持久化，保证数据的可靠性。之后数据也会被写入到内存中，叫Memtable
		* Commit Log记录每次写请求的完整信息，此时并不会根据主键进行排序，而是顺序写入，这样进行磁盘操作时没有随机写导致的磁盘大量寻道操作，对于提升速度有极大的帮助
		* Commit Log会在Memtable中的数据刷入SSTable后被清除掉，因此它不会占用太多磁盘空间
		* 写入到Memtable时，Cassandra能够动态地为它分配内存空间，你也可以使用工具自己调整。
	2. memtable满了之后会将数据刷入数据文件SSTable，并清空commit log. 
		* 当达到阀值后，Memtable中的数据和索引会被放到一个队列中，然后flush到磁盘，可以使用memtableflushqueue_size参数来指定队列的长度。当进行flush时，会停止写请求。
		* 顺序磁盘IO操作，降低大量写操作对存储系统的压力
		* 一个commit log对应多个sstable. 
	3. 根据consistency level, 刷入SSTable后回应协调者. 
	4. 每个table包含多个SSTable和Memtable
		* 为什么要分成多个？减小表的大小来提高sequential scan的效率
		* 难道不用检查多个sstable么？ bloom filter极大地提高效率
	
