# tomcat-redis-session-manager

> 使用redis配置tomcat共享session

### 结构图：

<img src="http://images.cnitblog.com/blog/536814/201501/301356402377480.png"/>

### 分析：

	分布式web server集群部署后需要实现session共享，针对 tomcat 服务器的实现方案多种多样，
	比如 tomcat cluster session 广播、nginx IP hash策略、nginx sticky module等方案，
	本文主要介绍了使用 redis 服务器进行 session 统一存储管理的共享方案。

### 必要环境：

* java1.7
* tomcat7



## 步骤

1. 添加redis session集群依赖的jar包到 TOMCAT_BASE/lib 目录下

	* tomcat-redis-session-manager-2.0.0.jar
	* jedis-2.5.2.jar
	* commons-pool2-2.2.jar


2. 修改 TOMCAT_BASE/conf 目录下的 context.xml 文件

		<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
		<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
         host="localhost"
         port="6379"
         database="0"
         maxInactiveInterval="60"
         sessionPersistPolicies="PERSIST_POLICY_1,PERSIST_POLICY_2,.."
         sentinelMaster="SentinelMasterName"
         sentinels="sentinel-host-1:port,sentinel-host-2:port,.."/>

	属性解释：

	*	**host** 						redis服务器地址
	*	**port** 						redis服务器的端口号
	*	**database** 					要使用的redis数据库索引
	*	**maxInactiveInterval** 		session最大空闲超时时间，如果不填则使用tomcat的超时时长，一般tomcat默认为1800 半个小时
	*	**sessionPersistPolicies**		session保存策略，可选

			[SAVE_ON_CHANGE]: every time session.setAttribute() or session.removeAttribute() is called the session will be saved. Note: This feature cannot detect changes made to objects already stored in a specific session attribute. Tradeoffs: This option will degrade performance slightly as any change to the session will save the session synchronously to Redis.

			[ALWAYS_SAVE_AFTER_REQUEST]: force saving after every request, regardless of whether or not the manager has detected changes to the session. This option is particularly useful if you make changes to objects already stored in a specific session attribute. Tradeoff: This option make actually increase the liklihood of race conditions if not all of your requests change the session.
	* **sentinelMaster**		redis集群主节点名称
	* **sentinels**				redis集群列表配置

3. 重启tomcat，session存储即可生效