
### Distributed ID Generator

1. __场景__
	1. 直观：
		* 用户 id
		* 聊天信息 id
		
2. __需求__
	* 非数字性需求
		1. id作为数据库的主键, 需要保证 __全局唯一__, 数据库会在这个字段上建立  clustered index. 
		2. id要尽可能短，以方便检索以及节省内存. 
		3. 
