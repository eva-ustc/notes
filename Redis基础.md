# Redis基础
## 数据类型
### strings
* `set key value`和`get key`，注意set操作会覆盖已有的key/value，若不希望覆盖可使用`set key value nx`
* `mset key1 value1 key2 value2 ...`和`mget key1 key2 ...`一次性完成多个key/value关系
* `incr key`加一，`incrby key increment`加increment
* `decr key`减一，`decr key decrement`减decrement
### Lists
* `lpush key value...`将多个value依次插入到key的最左边，`rpush`最右
* `lrange key start stop`输出key的start到stop范围，负数表示倒数
* `lpop key`和`rpop key`弹出key的最左或最右
* `lset key index value`将key的index位置元素**修改**为value
### Hashes
* `hset key field value`和`hmset key field1 value1 field2 value2...`
* `hget key field`和`hmget key field1 field2...`
* `hincrby key field increment`
### 无序集合
* `sadd key member1 member2 ...`
* `smembers key`输出所有元素
* `sismember key member`元素member是否在key内
### 有序集合
* `zadd key score1 member1 score member2...`
* `zrange key start stop`正序输出，`zrevrange key start stop`逆序输出，在末尾加上参数`withscores`可以同时输出记录值。
* `zincrby key increment member`
## 系统管理
### 适用于全体类型的常用命令
* `exists key1 key2...`判断多个key是否存在
* `del key1 key2`删除多个key
* `type key`返回key的类型(none, string, list, hash, set, zset)不存在返回none
* `keys pattern`根据pattern筛选并打印出已有的key
* `randomkey`随机返回一个已存在的key，若没有则返回`(nil)`
* `clear`清屏
* `rename key newkey`重命名，若newkey存在则覆盖
* `renamenx key newkey`重命名，若newkey存在则取消
* `dbsize`返回当前存在的key个数
### 时间相关命令
* `expire key seconds`key在seconds秒后自动删除
* `ttl key`查看key剩余生存时间
* `flushdb`删除当前数据库的所有key，`flushall`删除所有数据库的所有key
### 设置相关命令
* `config get`用来读取运行Redis服务器的配置参数，`config set`用于更改运行Redis服务器的配置参数。
* `auth`认证密码
* `config resetstat`重置数据统计报告
### 查询信息
* `info [section]`查询redis相关信息
    1. server: Redis server的常规信息
    1. clients: Client的连接选项
    1. memory: 存储占用相关信息
    1. persistence: RDB and AOF 相关信息
    1. stats: 常规统计
    1. replication: Master/slave请求信息
    1. cpu: CPU 占用信息统计
    1. cluster: Redis 集群信息
    1. keyspace: 数据库信息统计
    1. all: 返回所有信息
    1. default: 返回常规设置信息
    * 若命令参数为空，info命令返回所有信息。
## 高级应用
* `config set requirepass 密码`设置密码，`auth 密码`认证
* `multi`进入事务上下文，直到`exec`。执行过程中出现错误不会回滚。
* 持久化机制
* 虚拟内存的使用
