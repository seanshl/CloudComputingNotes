### Articles
* [Cap Theorem Revisited](http://robertgreiner.com/2014/08/cap-theorem-revisited/)
* [Proof of CAP](http://mwhittaker.github.io/2014/08/16/illustrated-proof-cap-theorem/)
### Cap Theorem
1. 基本定义
	* 在一个分布式系统中: **Consistency, Availability和Partition Tolerance**三者中只能满足二者.
	* **Consistency**: read is guaranteed to return the most recent write for a given client.
	* **Availability**: A non-failing node will return a reasonable response within a reasonable amount of time (no error or timeout).
	* **Partition Tolerance**: The system will continue to function when network partitions occur.
	* 分布式系统中failure概率成倍增长，所以在分布式系统中partition tolerance是必须被保证的. 
