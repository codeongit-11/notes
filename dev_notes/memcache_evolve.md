...menustart

 - [缓存进化之路](#dd99991793bc97b139eb761cda7decef)
     - [单机时代](#ce92167adfc1761d98ad9bfaa76e5d2c)
     - [多机分片(sharding)](#2dbb8fd1f965f9a201a44386a5720311)
     - [代理方案](#919b612698ed63acc1a4deaf6c12f026)
     - [分组多写方案](#d9db3ea95b296a3b197864f46634a071)
     - [分布式缓存](#8942654439694409860e460c8d4e5a79)
     - [Couchbase的实现](#9bb1db046a3075981b4d40b0ba44ee42)
         - [vBucket](#ac28335cd43ee4c05fbdca3df08dd089)

...menuend


<h2 id="dd99991793bc97b139eb761cda7decef"></h2>

# 缓存进化之路

<h2 id="ce92167adfc1761d98ad9bfaa76e5d2c"></h2>

## 单机时代

 - 一切都是美好的,缓存只是为了解决磁盘访问速度问题
 - 基本上就是个HashMap
 - 当单机纵向扩展(Scaling Up)遇到瓶颈时,必须探寻横向扩展(Scaling out)的方案.

<h2 id="2dbb8fd1f965f9a201a44386a5720311"></h2>

## 多机分片(sharding)

 - 横向扩展最简单的方式就是分片,无论存储还是缓存.
 - 分片最简单的方式就是 `node=nodes[hash(key)/%nodes.length]`, n表示机器的节点数
    - 但这样的方式, 当新增加节点或者减少节点的时候,n变化基本上会导致所有的key对应的节点都发生变化,造成缓存震荡
 - 另外一种分片就是静态key分片,应用配置中给每个节点设置静态hash区间,解决缓存震荡的问题.
    - 带来的问题是如果某个节点宕机,不能自动摘除,无法实现高可用。
 - 一致性哈希(Consistent Hashing) 
    - 避免了减少或者增加节点导致的缓存震荡
    - 一致性哈希的漂移机制一定程度上实现了高可用(相对静态分片) 
    - 但也存在以下问题: 
        - 如果请求量非常大,宕机后1/n的数据穿透也不可接受
        - 下线或者新增节点比较麻烦,需要应用修改配置升级,同时也会导致上面的问题
        - 无法动态增减节点

<h2 id="919b612698ed63acc1a4deaf6c12f026"></h2>

## 代理方案

 - 通过代理实现高可用也是一种可选方式 ,具体可参看 Redis HA 方案选型 
 - 存在的问题:
    - 性能瓶颈,增加运维复杂度,不能很好解决单点问题(Redis代理一般要依赖Redis的主从复制机制,memcached代理无法实现分组多写)

<h2 id="d9db3ea95b296a3b197864f46634a071"></h2>

## 分组多写方案

 - 同一个memcached集群,配置多组,写的时候多写,读的时候随机读或者根据配置规则读.
 - 一组中读取不到,则读取另一组.
 - 虽然有点笨,但也能解决集群方案尚未成熟的时候的缓存的高可用问题
 - 存在以下问题:
    - 应用层的配置比较复杂,上线下线节点更加麻烦
    - 写的时候要多写,对性能有影响
    - 纯内存型缓存,如果数据量比较大,内存数据丢失(重启服务等),热缓存的成本比较高

<h2 id="8942654439694409860e460c8d4e5a79"></h2>

## 分布式缓存
 
 - 我们期望中的理想缓存:
    - 自动分片,不需要应用关心 
    - failover,高可用,无单点问题 
    - 可动态扩容,可自动迁移数据实现均衡 
    - 自动复制(Replication),无需应用通过多写解决
    - 可持久化
 - 以上需求就是分布式缓存需要解决的问题

<h2 id="9bb1db046a3075981b4d40b0ba44ee42"></h2>

## Couchbase的实现

 - 要解决自动分片以及动态扩容问题,首先面临的问题就是客户端怎么知道数据存到那个节点上了.
 - 因为这个是动态变化的,  猜测可以用个中心节点去保存数据和节点的映射关系
 - 这个方式在分布式文件系统中用的多,但用在缓存上明显不合适了,因为缓存的数据都是小数据,这样中心节点就成瓶颈了,但思路可以借鉴.
 - Couchbase引入了vBucket的概念

<h2 id="ac28335cd43ee4c05fbdca3df08dd089"></h2>

### vBucket

![](https://raw.githubusercontent.com/mebusy/notes/master/imgs/memcache_evolve_vbucket.png)

 - vBucket其实就是key的分组,然后配置中心只需维护vBucket到节点的映射关系
 - 复制以及数据迁移都以vBucket为单位,这样降低了复制和数据迁移的成本
 - 原来复制迁移数据,需要将整个节点的数据都迁移完成后才能提供服务,而有了vBucket的概念后,一个vBucket迁移完即可提供对该vBucket下的数据的访问服务 
    - 注:Couchbase中的vBucket和Bucket是不同的概念.Bucket是Couchbase的数据分区,相当于虚拟的数据库集群.二者没有任何关系
 - TAP协议
    - memcached本身没有提供数据同步机制,于是Couchbase引入了TAP协议来实现数据复制





