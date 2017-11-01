### Rest
1. **理解**
	1. **REST**：(Resource Representational State Transfer). 资源表现层状态转移
	
2. **Restful API**
	1. URL中**只使用名词**来指定资源，原则上不使用动词。
	2. 用HTTP协议里的**动词来**实现资源的**添加，修改，删除**等操作。即通过HTTP动词来实现资源的状态扭转：
		* GET 用来获取资源，
		* POST 用来新建资源（也可以用于更新资源），
		* PUT 用来更新资源，
		* DELETE 用来删除资源。
3. __Restful API 设计__
	1. 协议：始终使用__https__.
	2. 版本：api版本号放在url中： ```https://api.example.com/v1/```
	3. endpoint: 表示api的具体url, 每个url代表一种资源， 所以url中不能有动词，只能有名字，而且所用的名词往往与数据库的表格名字对应，所以一般也是复数.
	4. 对于资源的具体操作，用http动词表示
		1. GET(select）
		2. POST (Create)
		3. PUT (Update) ： 客户端提供改变后的完整资源
		4. PATCH (Update) :客户端提供改变的属性
		5. DELETE (delete)
	5. eg
		* GET /zoos/ID/animals: 列出某个动物园的所有动物
		* DELETE: /zoos/ID/animals/ID: 删除某个动物园的指定动物
	6. Filtering
		* API应该体统参数进行过滤结果的筛选
		* ?limit=10：指定返回记录的数量
		* ?offset=10：指定返回记录的开始位置。
		* ?page=2&per_page=100：指定第几页，以及每页的记录数。
		* ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
		* ?animal_type_id=1：指定筛选条件
	7. Status Codes
		* 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
		* 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
		* 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
		* 204 NO CONTENT - [DELETE]：用户删除数据成功。
		* 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
		* 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
		* 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
		* 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
		* 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
		* 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
		* 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
		* 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
	

### Figures
1. google:
	* a rack -> 80 servers. 
	* a cluster -> 30 racks.
	
### Cap Theorem
1. (CAP Theorem: Revisited)[http://robertgreiner.com/2014/08/cap-theorem-revisited/]
