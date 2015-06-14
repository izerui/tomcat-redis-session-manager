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
* redis2.8



## 步骤

1. 添加redis session集群依赖的jar包到 TOMCAT_BASE/lib 目录下

	* <a href="https://github.com/izerui/tomcat-redis-session-manager/blob/master/jar/tomcat-redis-session-manager-2.0.0.jar?raw=true" target="_blank">tomcat-redis-session-manager-2.0.0.jar</a>
	* <a href="https://github.com/izerui/tomcat-redis-session-manager/blob/master/jar/jedis-2.5.2.jar?raw=true" target="_blank">jedis-2.5.2.jar</a>
	* <a href="https://github.com/izerui/tomcat-redis-session-manager/blob/master/jar/commons-pool2-2.2.jar?raw=true" target="_blank">commons-pool2-2.2.jar</a>


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
	*	**maxInactiveInterval** 		session最大空闲超时时间，如果不填则使用tomcat的超时时长，一般tomcat默认为1800 即半个小时
	*	**sessionPersistPolicies**		session保存策略，除了默认的策略还可以选择的策略有：
			
			[SAVE_ON_CHANGE]:每次 session.setAttribute() 、 session.removeAttribute() 触发都会保存. 
				注意：此功能无法检测已经存在redis的特定属性的变化，
				注意：这种策略会略微降低会话的性能，任何改变都会保存到redis中.

			[ALWAYS_SAVE_AFTER_REQUEST]: 每一个request请求后都强制保存，无论是否检测到变化.
				注意：对于更改一个已经存储在redis中的会话属性，该选项特别有用. 
				权衡：如果不是所有的request请求都要求改变会话属性的话不推荐使用，因为会增加并发竞争的情况。
	* **sentinelMaster**		redis集群主节点名称（Redis集群是以分片(Sharding)加主从的方式搭建，满足可扩展性的要求）
	* **sentinels**				redis集群列表配置(类似zookeeper，通过多个Sentinel来提高系统的可用性)

3. 重启tomcat，session存储即可生效