### Design a key-value cache to save the results of the most recent web server queries

1. __场景__
	* 用户发送一个请求，cache hit
	* 用户发送一个请求，cache miss
	* 高可用性
  
2. __限制与需求__
	* 非数字性需求
		1. traffic不会分布均匀，popular queries应该经常hit
		2. 需要方法来决定如何expire/refresh （cache common problem）
		3. cache中需要能够快速查找
		4. 限制cache的内存，提高查找速度 需要决定what to keep/remove, 
	* 数字性要求
		1. 10M users
		2. 10B queries / month
		3. cache millions queries
		4. query storage
			* query - 50 bytes
			* title - 20 bytes
			* snippet - 200 bytes
			* total - 270 bytes
			* 2.7 T的大小，如果10B的query都是独一无二并且都被存储起来.
			* 270 bytes per search * 10B searches / month
			* 4000 requests per second
			
		5. 公式转换
			* 1 request per second = 2.5M requests per month
			* 400 requests per second = 1B requests per month
			
3. __应用__
	* High Level Design
		![顶层设计](https://camo.githubusercontent.com/57223dafbceaf008d0fcec518ff40932f504b985/687474703a2f2f692e696d6775722e636f6d2f4b715a336453782e706e67)
	
	 
	
			
		
