# Redis简介
## 1. Redis（全称：`Remote Dictionary Server` 远程字典服务）是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、`Key-Value数据库`，并提供多种语言的API
## 2. Redis是一个简单的,高效的,分布式的,基于内存的缓存工具.架设好服务后,通过网络连接(类似于数据库),提供key-Value式缓存服务.简单是Redis突出的特色.简单可以保证核心功能的稳定和优异.
## 3. 总结
* Redis单个key存入512M大小
* Redis支持多种类型的数据结构(String,list,hash,set,zset)
* Redis是单线程 原子性
* Redis可以持久化 因为使用了RDB和AOF机制
* Redis支持集群 而且Redis支持库(0-15)16个库
* Redis还可以做消息队列 如聊天室 IM
## 4. 优点
* 丰富的数据结构
* 高速读写,Redis使用自己实现的分离器,代码量极短,没有用lock(MySQL),因此效率非常高
## 5.缺点
* 持久化.Redis直接将数据库储存到内存中,要将数据保存到磁盘上,Redis可以使用两种方式实现持久化过程.定时快照(snapshot):每隔一段时间将整个数据库写到磁盘上,每次均是写全部数据,代价非常高.第二种方式基于语句追加(aof):只追踪变化的数据,但是追加log可能过大,同时所有的操作均重新执行一遍,回复速度慢.
* 耗内存,占用内存过高.
# 安装
## Linux环境安装
Redis是C语言开发,安装Redis需要先将官网下载的源码进行编译,编译需要依赖gcc的环境,如果没有gcc,需要先安装gcc.  
首先需要root用户登录,如果不是root用户输入`su root`来切换用户,还有就是Linux需要连接外网,执行`yum -y install gcc automake autoconf libtool make`来安装gcc环境  
注意:运行yum时出现`/var/run/yum.pid`已被锁定,pid为xxxx的另一个程序正在运行的问题解决方案:`rm -f /var/run/yum.pid`  
环境安装成功后开始安装Redis  
* 下载安装文件`wget http://download.redis.io/releases/redis-4.0.1.tar.gz`
* 解压文件`tar zxvf redis-5.0.7.tar.gz`
* 进入文件目录`cd redis-5.0.7/`
* 编译`make MALLOC=libc`
* 安装`make PREFIX=/usr/local/redis install`
# 启动
## 服务端
进入到安装目录执行`./bin/redis-server`
## 客户端
进入到安装目录执行`./bin/redis-cli`
## 检测是否能连接到服务端
启动客户端后执行`ping`
# 配置
1. 复制安装包里的`redis.conf`文件到安装目录,和bin目录同级`cp redis.conf /usr/local/redis`
2. 打开配置文件`vim redis.conf`

## 配置文件说明

* `daemonize no`:redis默认不是以守护进程运行的,可以通过修改daemonize no来启动守护进行
* `port 6379`:redis默认端口号
* `bind 127.0.0.1`:绑定的主机地址(可以将它给注释掉)
* `timeout 300`:当客户端闲置多长时间后关闭连接,如果为0,表示关闭该功能
* `loglevel verbose`:日志记录级别,redis共有四个级别`debug,verbose,notice,warning`,默认为verbose
* `logfile stdout`:日志记录方式,默认为标准输出,如果配置redis为守护进程方式运行,日志记录方式为标准输出,则日志会发送给/dev/null
* `databases 16`:数据库的数量,默认数据库为0,可以使用SELECT <dbid>命令在连接上指定数据库id
* `save <seconds> <changes>`:指定在多长时间内,有多少次更新操作,就将数据同步到数据文件,可以多个文件配合
    * `save 900 1`:900秒内有1个更改
    * `save 300 10`300秒内有10个更改
    * `save 60 10000`60秒内有10000个更改
* `rdbcompression yes`:指定储存至本地数据库时是否压缩数据,默认为yes,redis采用LZF压缩,如果为了节省CPU时间,可以关闭该选项,但会导致数据库文件变的巨大
* `dbfilename dump.rdb`:指定本地数据库文件名称,默认值为dump.rdb
* `dir ./`:本地数据库文件存放目录
* `slaveof <masterip> <masterport>`:设置当本机为slave服务时,设置master服务的IP地址及端口,在redis启动时,它会自动从master进行数据同步
* `masterauth <master-password>`:当master服务设置了密码保护时,slave服务连接master的密码
* `requirepass foobared`:设置redis连接密码,如果配置了连接密码,客户端在连接redis时需要通过AUTH<password>命令提供密码,默认关闭,如果通过java连接的话就必须要设置
* `maxclients 128`:设置同一时间最大客户端连接数,默认是无限制的,redis可以同时打开的客户端连接数为redis进程可以打开的最大文件描述符数,如果设置为0,表示不做限制,当客户端连接数到达限制时redis会关闭新的连接并向客户端返回`max number of clients reached`错误信息
* `maxmemory <bytes>`:设置redis最大内存限制,redis在启动时会把数据加载到内存中,达到最大内存后,redis会先尝试清除已到期或即将到期的key,当此方法处理后,仍然达到最大内存设置,将无法再进行写入操作,但仍可以进行读取操作.redis新的vm机制,会把key存放到内存中,value存放到swap区
* `appendonly no`:指定是否没每次更新操作后进行日志记录,redis在默认情况下是异步的把数据写入磁盘,如果不开启,可能会在断电时导致一段时间的数据丢失.因为redis本身同步数据文件是按上面save条件来同步的,所以有的数据会在一段时间内只存在于内存中.默认为no
* `appendfilename appendonly.aof`:更新日志文件名,默认appendonly.aof
* `appendfsync everysec`:更新日志条件
    * `no`:等操作系统进行数据缓存同步到磁盘(快)
    * `always`:表示每次更新操作后手动调用fsync()将数据写入到磁盘(慢,安全)
    * `everysec`:每秒同步一次(折中,默认值)
* `vm-enabled no`:指定是否启用虚拟机内存机制,默认值为no,vm机制将数据分页存放,由redis将访问量较少的页即冷数据swap到磁盘上,访问多的页面由磁盘自动换出到内存中.
* `vm-swap-file /tmp/redis.swap`:虚拟内存文件路径,默认值为/tmp/redis.swap,不可多个redis实例共享
* `vm-max-memory 0`:将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(redis的索引数据 就是keys),当vm-max-memory设置为0的时候其实是所有value都存在于磁盘.默认为0
## 内存溢出解决方案
1. 为数据设置超时时间或设定内存空间
2. 采用LRU算法动态将不用的数据删除.内存管理的一种页面置换算法,对于在内存中但又不用的数据块(内存块)叫做LRU,操作系统会根据那些数据是属于LRU而将其移出内存而腾出空间来加载另外的数据.
    * `volatile-lru`:设定超时时间的数据中,删除最不常使用的数据
    * `allkeys-lru`:查询所有的key中最近最不常使用的数据进行删除,这是应用最广泛的策略
    * `volatile-random`:在已经设定了超时时间的数据中随机删除
    * `allkeys-random`:查询所有的key,之后随机删除
    * `volatile-ttl`:查询全部设定超时时间的数据,之后排序,将马上将要过期的数据进行删除操作
    * `noeviction`:不会进行删除操作,如果内存溢出则报错返回
    * `volatile-lfu`:从所有配置了过期时间的键中驱逐试用频率最少的键
    * `allkeys-lfu`:从所有键中驱逐试用频率最少的键
## 配置完成
服务端启动时的命令变成`./bin/redis-server ./redis.conf`  
查看是否启动成功`ps -ef | grep -i redis`
客户端启动命令`redis-cli -h IP地址 -p 端口 -a 密码`
# 关闭
1. 断电,非正常关闭.容易数据丢失
    * 查询PID `ps -ef | grep -i redis`
    * kill -9 PID
2. 正常关闭,数据保存
    * `./bin/redis-cli shutdown` 关闭redis服务,通过客户端进行shutdown
    * 如果redis设置了密码,需要先在客户端通过密码登录,再进行shutdown即可关闭服务端
# 常用命令
## key命令
* `DEL key`:在key存在的时候删除key
* `DUMP key`:序列化给定的key,并返回被序列化的值
* `EXISTS key`:检查给定的key是否存在
* `EXPIRE key seconds`:为给定的key设置过期时间(以秒计)
* `PEXPIRE key milliseconds`:设置key的过期时间以毫秒计
* `TTL key`:以秒为单位,返回给定key的剩余生存时间(TTL,time to live)
* `PTTL key`:以毫秒为单位,返回key的剩余过期时间
* `PERSIST key`:移出key的过期时间,key将持久保持
* `KEYS pattern`:查找所有符合给定模式(pattern)的key
    * `*`代表所有
    * `?`代表一个字符(keys 一部分key:?)
* `RANDOMkey`:从当前数据库随机返回一个key
* `RENAME key newkey`:修改key的名称
* `MOVE key db`:将当前数据库的key移动到给定的数据库db当中(move 要移动的key 0~15)
* `TYPE key`:返回key所储存的值得类型
## String命令
* `SET KEY_NAME VALUE`:redis set 命令用于设置给定key的值,如果key已经存储值,set就覆盖旧值,且无视类型
* `SETNX key value`:只有在key不存在的时候设置key的值,setnx (set if Not exists)
* `MSET key value[key value...]`:同时设置一个或多个key-value对
* `GET key_name`:获取给定key的值.如果key不存在,返回nil.如果key储存的值不是字符串类型,返回一个错误
* `GETRANGE key start end`:获取储存在指定key中字符串的子字符串.字符串的截取范围由start和end两个偏移量决定
* `GETBIT key offset`:对key所储存的字符串值,获取指定偏移量上的位(bit)
* `MGET key[key2...]`:获取所有(一个或多个)给定key的值
* `GETSET KEY_NAME VALUE`:设置指定key的值,并返回key的旧值,当key不存在时,返回nil
* `STRLEN key`:返回key所储存的字符串的长度
* `INCR KEY_NAME`:将key中储存的数字值增1.如果key不存在,那么key的值会被初始化为0,然后在执行incr操作
* `INCRBY KEY_NAME 增量值`:将key中储存的数字加上指定的增量值
* `DECR KEY_NAME`:将key中储存的数字值减1
* `DECRBY KEY_NAME 值`:将key中储存的数字减去指定的值
* `APPEND KEY_NAME VALUE`:为指定的key追加至末尾,如果不存在,为其赋值
## 哈希命令
* `HSET KEY FIELD VALUE`:为指定的key,设置fild/value
* `HMSET KEY FIELD VALUE [FIELD1 VALUE1...]`:同时将多个field-value对设置到哈希表key中
* `HGET KEY FIELD`:获取储存到hash中的值,根据field得到value
* `HMGET KEY FIELD [FIELD1...]`:获取key所有给定字段的值
* `HGETALL key`:返回hash表中所有的字段和值
* `HKETS key`:获取所有哈希表中的字段
* `HLEN key`:获取哈希表中字段的数量
* `HDEL KEY FIELD1 [FIELD2]`:删除一个或多个哈希表字段
* `HSETNX KEY FIELD VALUE`:只有在字段不存在时,设置哈希表的值
* `HINCRBY KEY FIELD INCREMENT`:为哈希表key中的指定字段的整数值加上增量increment
* `HINCRBYFLOAT KEY FIELD INCREMENT`:为哈希表key中的指定字段的浮点数值加上增量increment
* `HEXISTS KEY FIELD`:查看哈希表key中,指定的字段是否存在
## List命令
* `LPUSH key value1 [value2...]`:将一个或多个值插入到列表头部(从左侧添加)
* `RPUSH key value1 [value2...]`:在列表中添加一个或者多个值(从右侧添加)
* `LPUSHX key value`:将一个值插入到已存在的列表头部,如果列表不在,操作无效
* `RPUSHX key value`:一个值插入已存在的列表尾部(最右边),如果列表不在,操作无效
* `LLEN key`:获取列表长度
* `LINDEX key index`:通过索引获取列表中的元素
* `LRANGE key start stop`:获取列表指定范围内的元素
* `LPOP key`:移除并获取列表的第一个元素(从左侧删除)
* `RPOP key`:移除列表的最后一个元素,返回值为移除的元素(从右侧删除)
* `BLPOP key1 [key2...] timeout`:移除并获取列表的第一个元素,如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
* `BRPOP key1 [key2...] timeout`:移除并获取列表的最后一个元素,如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
* `LTRIM key start stop`:对一个列表进行修剪,让列表只保留指定区间内的元素,不在指定区间内的元素将被移除
* `LSET key index value`:通过索引设置列表元素的值
* `LINSERT key BEFORE|AFTER world value`:在列表的元素前或后插入元素
* `RPOPLPUSH source destination`:移除列表的最后一个元素,并将改元素添加到另一个列表并返回
* `BRPOPLPUSH source destination timeout`:从列表中弹出一个值,将弹出的元素插入到另外一个列表中并返回它,如果列表中没有元素会阻塞列表直到等待超时或发现可弹出元素为止
## Set命令
* `SADD key member1 [member2...]`:向集合中添加成员
* `SCARD key`:获取集合的成员数
* `SMEMBERS key`:返回集合中的所有成员
* `SISMEMBER key member`:判断member元素是否是集合key的成员
* `SRANDMEMBER key[count]`:返回集合中一个或多个随机数
* `SREM key member1 [member2]`:移除集合中一个或多个元素
* `SPOP key [count]`:移除并返回集合中的一个随机元素
* `SMOVE source destination member`:将member元素从source集合中移动到destination集合中
* `SDIFF key1 [key2...]`:返回给定所有集合的差集(显示左侧)
* `SDIFFSTORE destination key1 [key2...]`:返回给定所有集合的差集并存入destination中
* `SINTER key1 [key2...]`:返回给定所有集合的交集(共有数据)
* `SINTERSTORE destination key1 [key2...]`:返回给定所有集合的交集并存入destination中
* `SUNION key1 [key2...]`:返回所有给定的并集
* `SUNIONSTORE destination key1 [key2]`:所有给定集合的并集存储在destination集合中
## ZSet命令
* `ZADD key score1 member1 [score2 member2]`:向有序集合添加一个或多个成员,或者更新已存在成员的分数
* `ZCARD key`:获取有序集合的成员数
* `ZOUNT key min max`:计算在有序集合中指定区间分数的成员数
* `ZRANGE key member`:返回有序集合中指定成员的索引
* `ZRANGE key start stop [WITHSCORES]`:返回有序集合中指定区间内的成员,通过索引,分数从高到低
* `ZREM key member [member...]`:移除有序集合中的一个或多个成员
* `ZREMRANGEBYRANK key start stop`:移除有序集合中给定的排名区间的所有成员(第一名是0)
* `ZREMRANGEBYSCORE key min max`:移除有序集合中给定的分数区间的所有成员
## 发布订阅命令
* 订阅频道
    * `SUBSCRIBE channel [channel...]`:订阅给定的一个或多个频道的信息
    * `PSUBSCRIBE pattern [pattern...]`:订阅一个或多个符合给定模式的频道
* 发布频道
    * `PUBLISH channel message`:将消息发送到指定的频道
* 退订频道
    * `UNSUBSCRIBE [channel [channel...]]`:指退订指定的频道
    * `PUNSUBSCRIBE [pattern [pattern...]]`:退订所有给定模式的频道
## 多数据库命令
* `select 数据库索引`:数据库切换
* `move key 数据库索引`:移动数据到另外一个数据库
* `flushdb`:清除当前数据库的所有key
* `flushall`:清除整个redis的数据库的所有key
## 事务命令
* `DISCARD`:取消事务,放弃执行事务块内的所有命令
* `EXEC`:执行所有事务块内的命令
* `MULTI`:标记一个事务块的开始
* `UNWATCH`:取消WATCH命令对所有key的监视
* `WATCH key [key...]`:监视一个或多个key,如果在事务执行之前这个可以被其他命令所改动,name书屋将被打断
# Java连接Redis
## 1. 安装jar包
```xml
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.4.2</version>
    </dependency>
```
## 2.测试代码
```java
public static void main(String[] args) {
        //连接Redis服务器
        Jedis jedis = new Jedis("192.168.0.107", 6379);
        //权限认证
        String auth = jedis.auth("201218");
        System.out.println(jedis.ping());
        jedis.set("glh","哈哈");
        System.out.println(jedis.get("glh"));
    }
```
`如果连接超时的话,就是Linux端口都没有开放` 
* 服务端执行
    * 查看开放的端口`firewall-cmd --list-ports`
    * 设置开放端口`firewall-cmd --zone=public --add-port=6379/tcp --permanent`
    * 重启防火墙`firewall-cmd --reload`
## 3. javaJedis连接池
```java
        public static void main(String[] args) {
            //创建连接池 基本配置信息
            JedisPoolConfig pc=new JedisPoolConfig();
            //最大连接数
            pc.setMaxTotal(10);
            //最大的空闲数
            pc.setMaxIdle(1);
            String host="192.168.0.107";
            int port=6379;
            //连接池
            JedisPool jp= new JedisPool(pc,host,port);
            //连接Redis服务器
            Jedis jedis = jp.getResource();
            //权限认证
            String auth = jedis.auth("201218");
            jedis.set("glh","哈哈");
            System.out.println(jedis.get("glh"));
    }
```
### 优化
```java
public static void main(String[] args) {
    Jedis jedis=JedisPoolUtil.getJedis();
    jedis.set("glh","哈哈");
    System.out.println(jedis.get("glh"));
    JedisPoolUtil.close(jedis);
    hash();
}

public class JedisPoolUtil {
    private static JedisPool pool=null;
    static {
        //创建连接池 基本配置信息
        JedisPoolConfig pc=new JedisPoolConfig();
        //最大连接数
        pc.setMaxTotal(10);
        //最大的空闲数
        pc.setMaxIdle(1);
        String host="192.168.0.107";
        int port=6379;
        //连接池
        pool= new JedisPool(pc,host,port);
    }
    public static Jedis getJedis(){
        //连接Redis服务器
        Jedis jedis = pool.getResource();
        //权限认证
        String auth = jedis.auth("201218");
        return jedis;
    }
    public static void close(Jedis jedis){
        jedis.close();
    }
}
```
## Jedis完成对Hash类型操作
```java

    public static void hash(){
        Jedis jedis = JedisPoolUtil.getJedis();
        String key="hglh";
        if (jedis.exists(key)){
            Map<String, String> stringStringMap = jedis.hgetAll(key);
            System.out.println(stringStringMap.get("id")+stringStringMap.get("name")+stringStringMap.get("age")+stringStringMap.get("remark"));
        }else {
            jedis.hset(key,"id","1");
            jedis.hset(key,"name","glh");
            jedis.hset(key,"age","19");
            jedis.hset(key,"remark","男");
        }
        JedisPoolUtil.close(jedis);
    }
```
## RedisTemplate
### 简介
* Spring data提供了RedisTemplate模板  
* 它封装了redis连接池管理的逻辑,业务代码无需关心获取,释放连接逻辑  
* spring redis同时支持了redis,jredis,rjc客户端操作
### Spring整合redis
1. 导入jar
```xml
<dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>Junit</groupId>
        <artifactId>Junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.0</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
        <version>2.1.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.4</version>
        <scope>provided</scope>
    </dependency>
```
2. 对应实体bean进行序列化操作
```java
@Data
public class User implements Serializable {
    private static final long serialVersionUID = 2785330542918446733L;
    private Integer id;
    private String name;
    private String password;
    private String username;
    private Integer age;
}
```
3. 编写配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
     <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
         <property name="max WaitMillis" value="100"></property>
         <property name="maxIdle" value="1"></property>
         <property name="maxTotal" value="10"></property>
    </bean>

    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="hostName" value="192.168.2.181"></property>
        <property name="port" value="6379"></property>
        <property name="password" value="201218"></property>
        <property name="poolConfig" ref="jedisPoolConfig"></property>
    </bean>
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="jedisConnectionFactory"></property>
    </bean>
</beans>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <context:annotation-config></context:annotation-config>
     <context:component-scan base-package="cn.glh" ></context:component-scan>
</beans>
```
4. 测试
```java
    @Test
    public void t1(){
        ClassPathXmlApplicationContext classPathXmlApplicationContext=new ClassPathXmlApplicationContext("spring.xml","redis_spring.xml");
        UserService bean = classPathXmlApplicationContext.getBean(UserService.class);
        String glh = bean.getString("glh");
        System.out.println(glh);
    }
```
# Redis缓存与数据库一致性
* ## 实时同步
对强一致性要求比较高的,应该采取实时同步方案,即查询缓存查询不到再从db查询,保存到缓存;更新缓存时,先更新数据库,再将缓存的设置过期(建议不要去更新缓存内容,直接设置过期时间)
* ## 异步队列
对于并发程度较高的,可采用异步队列的方式同步,可采用kafka等消息中间件处理消息生产和消费
# 主从复制
`一个Redis服务可以有多个该服务的复制品,这个Redis服务称为master,其它复制称为slaves`

# 集群搭建
## 配置
1. 执行`mkdir /usr/local/redis_cluster`创建Redis节点安装目录
2. 在redis_cluster目录中创建7001~7006名称的文件夹
3. 将redis-conf分别拷贝到7001-7006文件夹下
4. 由于Linux上的redis处于安全保护模式这就让你无法从虚拟机外部去轻松建立连接,如果外部访问:redis.conf中设置保护模式为 `protected-mode no`
    * `bind 127.0.0.1 ::1`:绑定服务器IP地址
    * `port 7000`:绑定端口号,必须修改,以此来区别redis实例
    * `daemonize no`:是否在后台运行
    * `podfile /var/run/redis-7000.pid`:修改pid进程文件名,以端口号命名
    * `logfile /root/application/program/redis-cluster/70001/redis.log`:修改日志文件名称,以端口号命名
    * `dir /root/application/program/redis-cluster/70001`:修改数据文件存放地址,以端口号为目录名来区分
    * `cluster-enabled yes`:启动集群
    * `cluster-config-file nodes-7001.conf`:配置每个节点的配置文件
    * `cluster-node-timeout 15000`:配置集群节点的超时时间
    * `appendonly yes`:启动aof增强持久化策略
    * `appendfsync al`
5. 启动各个redis节点:
    * 将redis文件夹中的src文件拷贝到redis 7001-7006目录下`cp -r ./src /usr/local/redis_cluster/7001`
    * 进入redis集群配置文件目录下,依次启动`./7001/src/redis-server ./7001/redis.conf`
6. 检查redis启动情况  
    `ps -ef | grep -i redis`
## 创建集群
* redis官方提供了redis-tribe.rb这个工具,就在解压目录的src目录中,为了方便操作,将其文件复制到/usr/local/bin目录下,可以直接访问此命令
* 安装rvm` gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB`,再执行`curl -sSL https://get.rvm.io | bash -s stable`,再执行`find / -name rvm -print`,再执行`source /usr/local/rvm/scripts/rvm`,再执行 `rvm install 2.4.4`安装一个ruby版本,再使用一个ruby版本`rvm use 2.4.4`,再设置默认ruby版本`rvm use 2.4.4 --default`,查看ruby版本`ruby --version`,安装redis`gem install redis`
* 安装ruby`yum -y install ruby ruby-devel rubygems rpm-build`,再安装redis工具`gem install redis`