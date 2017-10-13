
### 设计短链接服务

1. **SNAKE原则**
	1. 场景与用例(想象与询问)
	2. 关键数据和数字(计算与询问)
	3. 应用架构设计,算法
	4. 存储方式
	5. 进化

1. **场景与用例**
	1. 创建新的短url(长-短/写): 用户输入长url,系统产生并存储短url.
		* expiration问题
		* 支持用户自定义的过期时间
	2. 读取短url(短-长/读): 用户点击短url, redirect to long url.
	3. 匿名用户
	4. service追踪统计用户数据
	5. service删除过期数据
	6. 高可用性.