# Redis

【参考尚硅谷Redis6课程】

## 1.Redis安装 

- 
  Redis下载

  [官网下载](https://download.redis.io/releases/redis-6.2.3.tar.gz?_ga=2.17544240.809792253.1621322316-754193776.1621322316)

- 将压缩包上传到Linux服务器上    

  ​		`/opt  目录下`    

- 解压压缩文件

  ​		`tar -zxvf redis-6.2.3.tar.gz`

- 安装GCC编译器

  ​		`yum install -y gcc`

- 查看GCC版本

  ​		`gcc --version`

- 进入解压后的redis文件夹

  `cd /opt/redis-6.2.3`

- 在redis-6.2.3目录下执行make命令

- 在redis-6.2.3目录下执行make install命令

- <font color=blue>若指定安装目录</font>，可将上两步替换为以下命令，将安装目录修改为自己的

  ​		`make PREFIX=/usr/local/redis install`

- ●默认安装目录

  ​		`/usr/local/bin`


[Redis配置文件详解](https://blog.csdn.net/suprezheng/article/details/90679790)

 

## 2.Redis启动 

- 前台启动(不推荐使用)

  `cd /usr/local/bin`

  `redis-server`

- ●后台启动

  - 拷贝一份redis.conf到其他目录

    `cp /opt/redis-6.2.3/redis.conf /etc/redis.conf`

  - 后台启动设置redis.conf的daemonize值

    - `vi /etc/redis.conf`
    - `no改成yes`  

  - <font color=blue>若用阿里云安装的redis，必须设置密码，不然会被挖矿</font>

    - <font color=blue>修改配置文件，设置requirepass值，即密码值</font>

      `requirepass xxx`

  - ○redis启动

    `cd /usr/local/bin`

    `redis-server /etc/redis.conf`

  - ○客户端访问

    - 无密码

      `redis-cli`	

    - 有密码

      `redis-cli -a 密码`

      或者使用`redis-cli`进入redis后，使用auth "密码" 认证

  - ○redis关闭

    `redis-cli shutdown`

##  3.Redis键(key)操作 

​			`set k1 a --->key:k1 ; value:a`

​			`set k2 b`

- `keys *` 		查看当前库所有key
- `exists key` 		判断某个key是否存在  -->exists k1
- `type key` 		查看你的key是什么类型  -->type k1 
- `del key` 	删除指定的key数据 -->del k1 
- `unlink	key`		根据value选择非阻塞删除仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作。
- `expire key 10` 	10秒钟：为给定的key设置过期时间 -->expire k2 10 
- `ttl key` 		查看还有多少秒过期，-1表示永不过期，-2表示已过期  -->ttl k2 	
- `flushdb` 	清空当前库

##  4.Redis数据类型 

redis到底有几种数据类型[传送查看](https://www.163.com/dy/article/G500BK4U053725XJ.html)

Redis命令手册，可自行下载
链接: https://pan.baidu.com/s/1KAabPz-WI7YurbmTEpusVw 提取码: m6xt 

 4.1 String 

 4.2 Hash 

 4.3 List 

 4.4 Set 

 4.5 Sorted Set 

 4.6 高级功能类型 

 4.6.1 Bitmaps 

 4.6.2 Geospatial 

 4.6.3 HyperLogLog  

##  5.Redis事务 

[传送查看](https://zhuanlan.zhihu.com/p/146865185)

##  6.Redis持久化 

###  6.1RDB 

[传送查看](https://www.cnblogs.com/yangming1996/p/12152090.html)

●优势

l  适合大规模的数据恢复

l  对数据完整性和一致性要求不高更适合使用

l  节省磁盘空间

l  恢复速度快

●劣势

l  Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

l  虽然Redis在fork时使用了写时拷贝技术,但是如果数据庞大时还是比较消耗性能。

l  在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

###  6.2AOF 

> [传送查看](https://www.cnblogs.com/yangming1996/p/12259931.html)

●优势

| 备份机制更稳健，丢失数据概率更低。

| 可读的日志文本，通过操作AOF稳健，可以处理误操作。

●劣势

| 比起RDB占用更多的磁盘空间。

| 恢复备份速度要慢。

| 每次读写都同步的话，有一定的性能压力。

| 存在个别Bug，造成恢复不能。

###  6.3 用哪个 

> 官方推荐两个都启用。
> 如果对数据不敏感，可以选单独用RDB。
> 不建议单独用 AOF，因为可能会出现Bug。
> 如果只是做纯内存缓存，可以都不用。

##  7.Redis主从复制 

###  7.1 环境配置 

- 新建myredis目录

  `[root@a etc]# mkdir myredis`

- 复制一份redis.conf到myredis目录（etc目录下复制过redis.conf， 如果没有就去安装目录复制）

  `[root@a etc]# cp redis.conf myredis/redis.conf`

- 修改myredis目录下redis.conf，将daemonize设置为yes,Appendonly 关掉

- myredis目录下新建redis6379.conf ,并添加内容

  `[root@a myredis]# vi redis6379.conf`

~~~bash
include redis.conf
pidfile /var/run/redis_6379.pid
port 6379
dbfilename dump6379.rdb 
~~~



- myredis目录下新建redis6380.conf ,并添加内容

  `[root@a myredis]# vi redis6380.conf`

~~~bash
include redis.conf
pidfile /var/run/redis_6380.pid
port 6380
dbfilename dump6380.rdb 
~~~



- myredis目录下新建redis6381.conf ,并添加内容

  `[root@a myredis]# vi redis6381.conf`

~~~bash
include redis.conf
pidfile /var/run/redis_6381.pid
port 6381
dbfilename dump6381.rdb 
~~~



- 关闭以前启动的redis服务

  `ps -ef | grep redis  查看端口号`

  `kill port`

- 使用三个配置文件分别启动redis

  `[root@a myredis]# redis-server redis6379.conf`

  `[root@a myredis]# redis-server redis6380.conf`

  `[root@a myredis]# redis-server redis6381.conf`

- 打开server-cli

  `[root@a myredis]# redis-cli -p 6379`

  `[root@a myredis]# redis-cli -p 6380`

  `[root@a myredis]# redis-cli -p 6381`

- 在客户端输入info replication打印主从复制的相关信息

  `127.0.0.1:6379> info replication`

  `127.0.0.1:6380> info replication`

  `127.0.0.1:6381> info replication`

- 配从库而不配主库，在从库中设置主从信息，选取6379为主

  `127.0.0.1:6380>  slaveof 127.0.0.1 6379`

  `127.0.0.1:6381>  slaveof 127.0.0.1 6379`

PS

~~~Plain Text
从机只能读数据而不能写数据
主机挂掉，重启就行，主从配置不会失效
从机重启需重设：slaveof 127.0.0.1 6379 
可以将配置增加到文件中。永久生效。
~~~



###  7.2 主从复制原理 

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621664167220-774ddca2-d714-472d-84ae-2242450a53be.png?x-oss-process=image%2Fresize%2Cw_534%2Climit_0)



###  7.3 一主二从 



![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621665459995-15dd7ab9-c970-415f-98d2-35660c447632.png)



- 主机宕机后，从机原地待命，不会成为主机

- 从机宕机后，重启后需要重新设置主机IP

- 从机宕机后，重新挂到主机上，会将主机的数据从<font color=blue>头开始复制</font>


###  7.4 薪火相传 

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621665539584-652db7b5-f00b-4978-b1fd-995d925a4d0b.png)



- 上一个Slave可以是下一个slave的Master，Slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master, 可以有效减轻master的写压力,去中心化降低风险。
- 用 slaveof  <ip><port>
- 中途变更转向:会清除之前的数据，重新建立拷贝最新的
- 风险是一旦某个slave宕机，后面的slave都没法备份

###  7.5 反客为主 

- 当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。

用slaveof  no one  将从机变为主机

###  7.6 哨兵模式 

- 调整为一主二仆模式，6379带着6380、6381

- 自定义的/myredis目录下新建sentinel.conf文件，名字绝不能错

  `[root@a myredis]# vi sentinel.conf`

- 配置哨兵


```
sentinel monitor mymaster 127.0.0.1 6379 1

PS: 其中mymaster为监控对象自定义的服务器名称， 1 为至少有多少个哨兵(启动的sentinel)同意从机切换为主机。
```

- 启动哨兵

  `[root@a myredis]# redis-sentinel  /myredis/sentinel.conf`

- 当主机挂掉，在从机中产生新的主机。原主机重启后会变为从机。

- 故障恢复	


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621667744458-a26861ac-3360-44cf-9e36-4850e2388d90.png)



> PS
> 		优先级在redis.conf中默认：replica-priority 100，值越小优先级越高
> 		偏移量是指获得原主机数据最全的
> 		每个redis实例启动后都会随机生成一个40位的runid

###  7.7 复制延时 

- 由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。


##  8 Redis集群 

Redis 集群实现了对Redis的<font color=blue>水平扩容</font>，即启动<font color=blue>N个redis节点</font>，将整个数据库分布存储在这N个节点中，每个节点存储<font color=blue>总数据的1/N</font>。

Redis 集群通过分区（partition）来提供一定程度的可用性（availability）：即使集群中有一部分节点失效或者无法进行通讯，集群也可以继续处理命令请求

###  8.1 环境配置 (aliyun版本) 

- 将myredis目录下的所有dump文件删除

  `rm -rf dump*`

- 创建六个redis配置文件


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621680889251-5cac6704-619e-405a-a311-c545c1d8c4f0.png)



每个文件的内容分别为

> redis6379.conf

~~~
include redis.conf
pidfile /var/run/redis_6379.pid
port 6379
dbfilename dump6379.rdb
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
~~~



> redis6380.conf

~~~
include redis.conf
pidfile /var/run/redis_6380.pid
port 6380
dbfilename dump6380.rdb
cluster-enabled yes
cluster-config-file nodes-6380.conf
cluster-node-timeout 15000
~~~



> redis6381.conf

~~~
include redis.conf
pidfile /var/run/redis_6381.pid
port 6381
dbfilename dump6381.rdb
cluster-enabled yes
cluster-config-file nodes-6381.conf
cluster-node-timeout 15000
~~~



> redis6389.conf

~~~
include redis.conf
pidfile /var/run/redis_6389.pid
port 6389
dbfilename dump6389.rdb
cluster-enabled yes
cluster-config-file nodes-6389.conf
cluster-node-timeout 15000
~~~



> redis6390.conf

~~~
include redis.conf
pidfile /var/run/redis_6390.pid
port 6390
dbfilename dump6390.rdb
cluster-enabled yes
cluster-config-file nodes-6390.conf
cluster-node-timeout 15000
~~~



> redis6391.conf

~~~
include redis.conf
pidfile /var/run/redis_6391.pid
port 6391
dbfilename dump6391.rdb
cluster-enabled yes
cluster-config-file nodes-6391.conf
cluster-node-timeout 15000
~~~



- 阿里云控制台添加安全组


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621681530918-b8cec623-b633-4985-8b44-78e61c33fb7e.png)

> <font color=red>PS ： 安全组的创建百度即可，注意安全组所属地区是否与你的服务器所在地区一致</font>



- 使用六个配置文件启动六个redis-server

  `[root@a myredis]# redis-server redis6379.conf`

  `[root@a myredis]# redis-server redis6380.conf`

  `[root@a myredis]# redis-server redis6381.conf`

  `[root@a myredis]# redis-server redis6389.conf`

  `[root@a myredis]# redis-server redis6390.conf`

  `[root@a myredis]# redis-server redis6391.conf`

  `[root@izwz94fh3gqfiee5nkg5gcZmyrdis`

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621681860617-14239187-ba4d-4e9e-9c80-9581c9a7f4ba.png)



- 确保所有redis实例启动后，nodes-xxxx.conf文件都生成正常


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621681981375-3e5c793c-38c2-4cc2-8677-a93c3a96f746.png)



- 进入下载的redis目录

  `cd  /opt/redis-6.2.3/src`

- 使用 redis-cli 创建整个 redis 集群（redis5.0版本之前使用的ruby脚本 redis-trib.rb，之后的版本已经集成该脚本）


```Bash
redis-cli --cluster create --cluster-replicas 1 8.129.52.78:6379 8.129.52.78:6380 8.129.52.78:6381 8.129.52.78:6389 8.129.52.78:6390 8.129.52.78:6391
```

> <font color=red>此处不要用127.0.0.1，请用真实IP地址，即服务器IP地址</font>
>
> <font color=red>--replicas 1 采用最简单的方式配置集群，一台主机，一台从机，正好三组。</font>



成功示例

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621682648692-dea7f4e8-c61d-4fad-9cec-880ce4c83bd4.png)



###  8.2 查看集群信息 

- -c 采用集群策略连接，设置数据会自动切换到相应的写主机

  `[root@a src]#  redis-cli -c -p 6379`

- 通过 cluster nodes 命令查看集群信息

  `127.0.0.1:6379> cluster nodes`

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621683570035-b8d06dd5-7d0c-4f97-8f45-675bc649810a.png)



###  8.3 cluster 如何分配当前六个节点 

- 一个集群至少要有三个主节点。

- 选项--cluster-replicas 1 表示我们希望为集群中的每个主节点创建一个从节点。

- 分配原则尽量保证每个主数据库运行在不同的IP地址(即不同的服务器)，每个从库和主库不在一个IP地址上。


###  8.4 什么是slots 

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621684958986-2d20ce93-69c7-4417-bc86-7041b232bde0.png)



一个 Redis 集群包含16384 个插槽（hash slot），数据库中的每个键都属于这16384 个插槽的其中一个，集群使用公式<font color=blue>CRC16(key) % 16384 来</font>计算键key 属于哪个槽，其中CRC16(key) 语句用于计算键key 的CRC16 校验和。集群中的每个节点负责处理一部分插槽。

​		举个例子，一个集群有<font color=blue>三个主节点</font>，其中：

​					节点 A 负责处理0号至5460号插槽。

​					节点 B 负责处理5461号至10922号插槽。

​					节点 C 负责处理10923号至16383号插槽。

###  8.5 向集群写值 

- 计算出slot后，自动重定向到对应集群节点


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621685492492-99d957f7-d747-4817-a3b1-f4d119cee03a.png)



- 通过{}定义组才可以使用mget,mset等多键操作


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621685670270-eb8a87d0-2a50-4cb6-a1fe-18b7064d6359.png)



###  8.5 查询集群中的值 

- cluster keyslot <key> ：计算键 key 应该被放置在哪个槽上

  `cluster keyslot k1`

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621685919817-de794104-a464-4078-9da0-6d5ef962ac63.png)



- cluster countkeysinslot <slot> ：返回槽 slot 目前包含的键值对数量。（槽slot的值必须在当前节点范围内否则返回0）


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621686622168-cba72737-5119-42e2-b599-0a57143969e3.png)



- cluster getkeysinslot <slot> <count> ：返回 count 个 slot 槽中的键 （槽slot的值必须在当前节点范围内否则返回empty array）


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621686735935-bb17e993-27bd-4303-b693-2e7b51e44120.png)



###  8.6 故障恢复 

- 某台<font color=blue>主机(A)</font>宕机后，它的<font color=blue>从机(B)</font>就会成为<font color=blue>主机(C)</font>，且再次启动宕机的<font color=blue>主机(A)</font>，<font color=blue>该主机(A)</font>会变成<font color=blue>C的从机</font>

- 如果某一段插槽的主从都挂掉，而cluster-require-full-coverage 为yes ，那么，<font color=blue>整个集群都挂掉</font>

- 如果某一段插槽的主从都挂掉，而cluster-require-full-coverage 为no ，那么，<font color=blue>该插槽</font>数据全都不能使用，也无法存储。

- redis.conf中的参数  cluster-require-full-coverage


###  8.7 集群好处与不足 

- 好处

  - 实现扩容

  - 分摊压力

  - 无中心配置相对简单


- 不足

  - 多键操作是不被支持的

  - 多键的Redis事务是不被支持的。lua脚本不被支持

  - 由于集群方案出现较晚，很多公司已经采用了其他的集群方案，而代理或者客户端分片的方案想要迁移至redis cluster，需要整体迁移而不是逐步过渡，复杂度较大


##  9 Redis应用问题(高并发场景) 

###  9.1 缓存穿透 

####  9.1.1 问题描述 

<font color=blue>key对应的数据在数据源并不存在</font>，此时若有<font color=blue>大量并发请求</font>过来，每次针对此key的请求从缓存获取不到，请求都会压到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621733790830-d7ffc000-0f06-4e07-8149-08495a1fb8a9.png)





####  9.1.2 解决方案 

一个一定不存在缓存及查询不到的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。

<hr/>

解决方案：

- <font color=blue> 对空值缓存</font>：如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，设置空结果的过期时间会很短，最长不超过五分钟
- <font color=blue>设置可访问的名单（白名单）</font>：使用bitmaps类型定义一个可以访问的名单，名单id作为bitmaps的偏移量，每次访问和bitmap里面的id进行比较，如果访问id不在bitmaps里面，进行拦截，不允许访问。
- <font color=blue>采用布隆过滤器</font>：(布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量(位图)和一系列随机映射函数（哈希函数）。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。)将所有可能存在的数据哈希到一个足够大的bitmaps中，一个一定不存在的数据会被这个bitmaps拦截掉，从而避免了对底层存储系统的查询压力。
- <font color=blue>进行实时监控</font>：当发现Redis的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务

###  9.2 缓存击穿 

####  9.2.1 问题描述 

<font color=blue>key(某个)对应的数据存在，但在redis中过期，</font>此时若有<font color=blue>大量并发请求</font>过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。



![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621734025111-91bbf46d-2bef-4429-b83d-8de010ebaadf.png)



####  9.2.2 解决方案 

key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题。

<hr/>

解决问题：

- <font color=blue>预先设置热门数据</font>：在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大这些热门数据key的时长
- <font color=blue>实时调整</font>：现场监控哪些数据热门，实时调整key的过期时长
- <font color=blue>使用锁</font>：
  - 就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db。

  - 先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX）去set一个mutex key

  - 当操作返回成功时，再进行load db的操作，并回设缓存,最后删除mutex key；当操作返回失败，证明有线程在load db，当前线程睡眠一段时间再重试整个get缓存的方法。


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621734045867-7a718986-99c5-4686-b4ee-b31d1e4ef69b.png)



###  9.3 缓存雪崩 

####  9.3.1 问题描述 

<font color=blue>key(大量)对应的数据存在，但在redis中过期，</font>此时若有<font color=blue>大量并发请求</font>过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

> PS.	缓存雪崩与缓存击穿的区别在于缓存雪崩<font color=blue>针对很多key缓存，缓存击穿则是某一个key</font>

####  9.3.2 解决方案 

缓存失效时的雪崩效应对底层系统的冲击非常可怕！

<hr/>

解决方案：

- <font color=blue>构建多级缓存架构</font>：nginx缓存 + redis缓存 +其他缓存（ehcache等）
- <font color=blue>使用锁或队列</font>：用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。不适用高并发情况
- <font color=blue>设置过期标志更新缓存</font>：记录缓存数据是否过期（设置提前量），如果过期会触发通知另外的线程在后台去更新实际key的缓存。
- <font color=blue>将缓存失效时间分散开</font>：比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

###  9.4 分布式锁 

####  9.4.1 问题描述 

随着业务发展的需要，原单体单机部署的系统被演化成分布式集群系统后，由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，单纯的Java API并不能提供分布式锁的能力。为了解决这个问题就需要一种跨JVM的互斥机制来控制共享资源的访问，这就是分布式锁要解决的问题！

<hr/>

分布式锁主流的实现方案：

1. 基于数据库实现分布式锁

2. 基于缓存（Redis等）

3. 基于Zookeeper


每一种分布式锁解决方案都有各自的优缺点：

1. 性能：redis最高

2. 可靠性：zookeeper最高


这里，我们就基于redis实现分布式锁。

####  9.4.2 解决方案 

redis命令（setnx）

<hr/>

- setnx加锁，del释放锁


> ​		加锁：setnx user niubi
> ​		释放锁: del user
>  <font color=red>问题：setnx刚好获取到锁，业务逻辑出现异常，导致锁无法释放</font>
>  <font color=red>解决：设置过期时间，自动释放锁。</font>



- 设置过期时间防止锁一直不被释放

> ​		加锁：setnx user niubi
> ​		设置过期：expire user 10
>  <font color=red>缺乏原子性：如果在setnx和expire之间出现异常，锁也无法释放</font>



- 同时设置锁和过期时间(推荐使用)

> ​		set user 10 nx ex 10
> <font color=red> 问题：可能会释放其他服务器的锁。</font>
>
>  场景：如果业务逻辑的执行时间是7s，锁过期时间为3s。执行流程如下
>
> 1. index1业务逻辑没执行完，3秒后锁被自动释放。
> 2. index2获取到锁，执行业务逻辑，3秒后锁被自动释放。
> 3. index3获取到锁，执行业务逻辑
> 4. index1业务逻辑执行完成，开始调用del释放锁，这时释放的是index3的锁，导致index3的业务只执行    1s就被别人释放。
>    最终等于没锁的情况。
>
>  <font color=red>解决：setnx获取锁时，设置一个指定的唯一值（例如：uuid）；释放前获取这个值，判断是否自己的锁</font>



- 设置UUID防误删(即将key的值设置为唯一值)

> ​		set user UUID nx ex 10
> <font color=red>问题：删除操作缺乏原子性。</font>
>
> 场景：
>
> 1. index1执行删除时，查询到的lock值确实和uuid相等
> 2. index1执行删除前，lock刚好过期时间已到，被redis自动释放
> 3. index2获取了lock开始执行方法
> 4. index1执行删除，此时会把index2的lock删除  （同一个锁）
>    index1 因为已经在方法中了，所以不需要重新上锁。index1有执行的权限。</
>
> <font color=red>解决：LUA脚本保证删除的原子性</font>



set 命令参数详解

> EX second ：设置键的过期时间为 second 秒。 SET key value EX second 效果等同于 SETEX key second value 
> PX millisecond ：设置键的过期时间为 millisecond 毫秒。 SET key value PX millisecond 效果等同于 							     
>   		PSETEX key millisecond value 。
> NX ：只在键不存在时，才对键进行设置操作。 SET key value NX 效果等同于 SETNX key value 。
> XX ：只在键已经存在时，才对键进行设置操作。



- setnx加锁，del释放锁

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621736779482-c68c8249-40e3-475e-86a7-ba049a9c4e45.png)





~~~java
@GetMapping("testLock")
public void testLock(){
    //1获取锁，setnx-->setIfAbsent
    Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "111");
    //2获取锁成功、查询num的值
    if(lock){
        Object value = redisTemplate.opsForValue().get("num");
        //2.1判断num为空return
        if(StringUtils.isEmpty(value)){
            return;
        }
        //2.2有值就转成成int
        int num = Integer.parseInt(value+"");
        //2.3把redis的num加1
        redisTemplate.opsForValue().set("num", ++num);
        //2.4释放锁，del
        redisTemplate.delete("lock");

    }else{
        //3获取锁失败、每隔0.1秒再获取
        try {
            Thread.sleep(100);
            testLock();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
~~~

> 1. 多个客户端同时获取锁（setnx）
> 2. 获取成功，执行业务逻辑{从db获取数据，放入缓存}，执行完成释放锁（del）
> 3. 其他客户端等待重试



<hr/>

- 同时设置锁和过期时间(推荐使用)


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621737077357-6f47c76e-8d16-4e46-b661-4845584cdf87.png)

~~~java
@GetMapping("testLock")
public void testLock(){
    //1获取锁，setnx-->setIfAbsent
    Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "111", 10, TimeUnit.SECONDS);
    //2获取锁成功、查询num的值
    if(lock){
        Object value = redisTemplate.opsForValue().get("num");
        //2.1判断num为空return
        if(StringUtils.isEmpty(value)){
            return;
        }
        //2.2有值就转成成int
        int num = Integer.parseInt(value+"");
        //2.3把redis的num加1
        redisTemplate.opsForValue().set("num", ++num);
        //2.4释放锁，del，保证锁必须被释放
        redisTemplate.delete("lock");  -->当业务执行时间小与过期时间时需要释放锁

    }else{
        //3获取锁失败、每隔0.1秒再获取
        try {
            Thread.sleep(100);
            testLock();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

~~~





- 设置UUID防误删


![image.png](https://cdn.nlark.com/yuque/0/2021/png/12805114/1621745985222-091ca300-14fa-4b32-bdba-f7a611075147.png)



~~~java
@GetMapping("testLock")
public void testLock(){
    //1获取锁，setnx-->setIfAbsent
    String uuid = UUID.randomUUID().toString();
    Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", uuid, 10, TimeUnit.SECONDS);
    //2获取锁成功、查询num的值
    if(lock){
        Object value = redisTemplate.opsForValue().get("num");
        //2.1判断num为空return
        if(StringUtils.isEmpty(value)){
            return;
        }
        //2.2有值就转成成int
        int num = Integer.parseInt(value+"");
        //2.3把redis的num加1
        redisTemplate.opsForValue().set("num", ++num);
        //2.4释放锁，del，保证锁必须被释放-->当业务执行时间小与过期时间时需要释放锁
        if(uuid.equals((String)redisTemplate.opsForValue().get("lock"))){
             redisTemplate.delete("lock"); -->删除自己的锁
        }

    }else{
        //3获取锁失败、每隔0.1秒再获取
        try {
            Thread.sleep(100);
            testLock();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

~~~



<hr/>

- LUA脚本保证删除的原子性


~~~java
@GetMapping("testLockLua")
public void testLockLua() {
    //1 声明一个uuid ,将做为一个value 放入我们的key所对应的值中
    String uuid = UUID.randomUUID().toString();
    //2 定义一个锁：lua 脚本可以使用同一把锁，来实现删除！
    String skuId = "25"; // 访问skuId 为25号的商品 100008348542
    String locKey = "lock:" + skuId; // 锁住的是每个商品的数据

    // 3 获取锁
    Boolean lock = redisTemplate.opsForValue().setIfAbsent(locKey, uuid, 3, TimeUnit.SECONDS);

    // 第一种： lock 与过期时间中间不写任何的代码。
    // redisTemplate.expire("lock",10, TimeUnit.SECONDS);//设置过期时间
    // 如果true
    if (lock) {
        // 执行的业务逻辑开始
        // 获取缓存中的num 数据
        Object value = redisTemplate.opsForValue().get("num");
        // 如果是空直接返回
        if (StringUtils.isEmpty(value)) {
            return;
        }
        // 不是空 如果说在这出现了异常！ 那么delete 就删除失败！ 也就是说锁永远存在！
        int num = Integer.parseInt(value + "");
        // 使num 每次+1 放入缓存
        redisTemplate.opsForValue().set("num", String.valueOf(++num));
        /*使用lua脚本解锁*/
        // 定义lua 脚本
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        // 使用redis执行lua执行
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
        redisScript.setScriptText(script);
        // 设置一下返回值类型 为Long
        // 因为删除判断的时候，返回的0,给其封装为数据类型。如果不封装那么默认返回String 类型，
        // 那么返回字符串与0 会有发生错误。
        redisScript.setResultType(Long.class);
        // 第一个要是script 脚本 ，第二个需要判断的key，第三个就是key所对应的值。
        redisTemplate.execute(redisScript, Arrays.asList(locKey), uuid);
    } else {
        // 其他线程等待
        try {
            // 睡眠
            Thread.sleep(1000);
            // 睡醒了之后，调用方法。
            testLockLua();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

~~~



####  9.4.3 总结 

1.加锁

~~~java
// 1. 从redis中获取锁,set k1 v1 px 20000 nx
String uuid = UUID.randomUUID().toString();
Boolean lock = this.redisTemplate.opsForValue().setIfAbsent("lock", uuid, 2, TimeUnit.SECONDS);

~~~



2.解锁

~~~java
// 2. 释放锁 del
String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
// 设置lua脚本返回的数据类型
DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
// 设置lua脚本返回类型为Long
redisScript.setResultType(Long.class);
redisScript.setScriptText(script);
redisTemplate.execute(redisScript, Arrays.asList("lock"),uuid);

~~~



3.重试

~~~java
Thread.sleep(500);
testLock();
~~~



> 为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下四个条件：
> \- 互斥性。在任意时刻，只有一个客户端能持有锁。
> \- 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
> \- 解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。
> \- 加锁和解锁必须具有原子性。

<hr/>

##  10 Redsi6.0新功能 

- ACL


> Redis ACL是Access Control List（访问控制列表）的缩写，该功能允许根据可以执行的命令和可以访问的键来限制某些连接。
>
> 在Redis 5版本之前，Redis 安全规则只有密码控制还有通过rename 来调整高危命令比如 flushdb , KEYS*, shutdown 等。Redis 6 则提供ACL的功能对用户进行更细粒度的权限控制



- IO多线程

> Redis 的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。之所以这么设计是不想因为多线程而变得复杂，需要去控制 key、lua、事务，LPUSH/LPOP 等等的并发问题。
>
> 多线程IO默认也是不开启的，需要在配置文件(redis.conf)中配置
> 			io-threads-do-reads  yes 
> 			io-threads 4



- 工具支持 Cluster

> 老版本Redis想要搭集群需要单独安装ruby环境，Redis5开始 将 redis-trib.rb 的功能集成到 redis-cli 。另外官方 redis-benchmark 工具开始支持 cluster 模式了，通过多线程的方式对多个分片进行压测。



- RESP3协议

> 新的 Redis 通信协议：优化服务端与客户端之间通信

- Client side caching客户端缓存


> 基于 RESP3 协议实现的客户端缓存功能。为了进一步提升缓存的性能，将客户端经常访问的数据cache到客户端。减少TCP网络交互。



- Proxy集群代理模式

> Proxy 功能，让 Cluster 拥有像单实例一样的接入方式，降低大家使用cluster的门槛。不过需要注意的是代理不改变 Cluster 的功能限制，不支持的命令还是不会支持，比如跨 slot 的多Key操作。



- Modules API

> Redis 6中模块API开发进展非常大，因为Redis Labs为了开发复杂的功能，从一开始就用上Redis模块。Redis可以变成一个框架，利用Modules来构建不同系统，而不需要从头开始写然后还要BSD许可。Redis一开始就是一个向编写各种系统开放的平台。