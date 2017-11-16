
### References
* [What is MapReduce](https://www.guru99.com/introduction-to-mapreduce.html)
* [快速理解MapReduce](http://blog.csdn.net/suifeng3051/article/details/41651851)


### 原理
* **存储结构**： 
	* key-value pair
* **用户参与Phases**:
	1. Map, 程序员必须定义map函数
	2. Reduce 程序员必须定义reduce函数
* **过程图**
	* ![](https://cdn.guru99.com/images/Big_Data/061114_0930_Introductio1.png)
	* ![](https://i.stack.imgur.com/EH8tY.jpg)
* **phases**
	1. 顶层调度
		* 将作为input的任务按照某种方式划分成M块，例如将文件分成M个数据片段
		* 每个数据片段大小从16M到64M不等
		* 用户程序在集群中创建大量副本
	2. master分配
		* Master主导任务在集群中的分配，有M个Map任务和R个reduce任务将会被分配. 
		* Master将一个Map任务或者一个reduce任务分配给一个空闲的worker. 
	3. Input Splits (input files -> map)
		* 此时input file被分成了M个部分		
		* 被分配map任务的worker读取相关数据片段，并使用用户定义的map函数生成并输出中间key/value pair, 缓存在**内存**中
	4. 存储中间值 (map -> intermediate) 
		* worker的缓存中的中间key/value通过分区函数分成R个区域，之后周期性地写入本地磁盘.缓存的key/value pair在本地磁盘上的存储位置将会被回传给master,由master负责将这些存储位置传给Reduce worker. 
	5. shuffle
		* reduce worker接收到master发来的中间值的存储位置后，使用**RPC**从map worker所在的主机磁盘上读取这些缓存为止. 当reduce worker读取了所有的中间数据后，通过对key进行排序后使得具有u相同key值的数据聚合在一起.这一不可能需要进行外部排序
	6. reduce
		* reduce worker遍历排序后的中间数据，对于每一个唯一的中间key值，reduce worker将这个值和它相关的中间value值传递给用户定义的reduce函数中去.reduce函数的输出被追加到所属分区的输出文件
		
		
 
