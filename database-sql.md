### RDBMS

1. **ACID**
	1. **Atomic**: 每个事务内部所有操作要么全部完成，要么全部不完成
	2. **Consistency:** 任何事务都使数据库从一个有效的状态转换到另一个有效状态
	3. **Isolation**: 并发执行事务的结果与顺序执行事务的结果相同
	4. **Persistency**: 事务提交后，对系统的影响是永久的
2. **包含的技术**  
	1. **Master-Slave Replication**  
  		* 主库同时负责读取和写入操作，并复制写入到一个或多个从库中，从库只负责读操作。树状形式的从库再将写入复制到更多的从库中去。如果主库离线，系统可以以只读模式运行，直到某个从库被提升为主库或有新的主库出现

	2. * ![](https://camo.githubusercontent.com/6a097809b9690236258747d969b1d3e0d93bb8ca/687474703a2f2f692e696d6775722e636f6d2f4339696f47746e2e706e67)  
      1. **Master-Master Replication**![](https://camo.githubusercontent.com/5862604b102ee97d85f86f89edda44bde85a5b7f/687474703a2f2f692e696d6775722e636f6d2f6b7241484c47672e706e67)

	3. 两个主库都负责读操作和写操作，写入操作时互相协调。如果其中一个主库挂机，系统可以继续读取和写入。

3. **Miscs**
	1. Clustered Index 
		table中数据的physical storage order 与 key的 logic order相同。也就是数据依据聚集索引，可以依次排列在存储硬件上. 一个表中只能有一个clustered index, 一般是做主键，提高查询效率。		
