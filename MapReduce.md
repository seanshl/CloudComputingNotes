
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
	1. input splits
		* 将作为input的任务按照某种方式划分成M块，例如将文件分成M个数据片段
		* 每个数据片段大小从16M到64M不等
		* 用户在集群中创建大量
		
 
