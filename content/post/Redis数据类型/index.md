---
title: "Redis数据类型"
date: 2023-04-12T00:23:11+08:00
draft: false
tags: ['Linux', 'Redis']
category: '学习之路'
---

# 五大数据类型

**Redis-key**

```shell
keys * # 查看所有的key

set key value # 设置key-value

exists key # 判断key是否存在

move key db编号 # 移除key

expire key 时间（秒） # 设置key过期时间

ttl key # 查看key剩余时间

type key # 查看key数据类型
```



## String（字符串）

```shell
127.0.0.1:6379> set key1 v1 # 设置值
OK
127.0.0.1:6379> get key1 # 获得值
"v1"
127.0.0.1:6379> keys * # 获得所有的key
1) "counter:__rand_int__"
2) "mylist"
3) "myhash"
4) "key1"
5) "key:__rand_int__"
127.0.0.1:6379> exists key1 # 判断key是否存在
(integer) 1
127.0.0.1:6379> append key1 "hello" # 向key后面追加内容（若当前key不存在，就相当于set key）
(integer) 7
127.0.0.1:6379> get key1 # 获得值
"v1hello"
127.0.0.1:6379> strlen key1 # 获得key的长度
(integer) 7
127.0.0.1:6379> append key1 ", hhhhh" # 向key后面追加
(integer) 14
127.0.0.1:6379> strlen key1 # 获得key长度
(integer) 14
127.0.0.1:6379> get key1 # 获得key
"v1hello, hhhhh"

```

```shell
127.0.0.1:6379> set views 0 
OK
127.0.0.1:6379> get views
"0"
127.0.0.1:6379> incr views # 类似于i++，自增1
(integer) 1
127.0.0.1:6379> get views
"1"
127.0.0.1:6379> incr views
(integer) 2
127.0.0.1:6379> decr views # 类似于i--，自减1
(integer) 1
127.0.0.1:6379> type views # 判断类型
string
127.0.0.1:6379> incrby views 10 # 增加步长，给key增加10
(integer) 11
127.0.0.1:6379> get views
"11"
127.0.0.1:6379> decrby views 5 # 减少步长，给key减少5
(integer) 6
127.0.0.1:6379> incr views
(integer) 7

```

```shell
127.0.0.1:6379> get key1
"v1hello, hhhhh"
127.0.0.1:6379> getrange key1 0 7 # 取得key的[0, 7]的字符串
"v1hello,"
127.0.0.1:6379> getrange key1 0 -1 # 取得整个字符串，和get key是一样的
"v1hello, hhhhh"
127.0.0.1:6379> setrange key1 0 hh # 设置从指定位置开始，替换后面字符串
(integer) 14
127.0.0.1:6379> getrange key1 0 -1
"hhhello, hhhhh"
```

```shell
127.0.0.1:6379> setex key2 30 "redis!" # 设置key并且设置过期时间
OK
127.0.0.1:6379> ttl key2
(integer) 26
127.0.0.1:6379> setnx key3 "Redis" # 若key不存在则创建key
(integer) 1
127.0.0.1:6379> get key3
"Redis"
127.0.0.1:6379> setnx key3 "MongoDB" # 若key不存在，则创建，若存在则返回0，不更改内容
(integer) 0
127.0.0.1:6379> get key3
"Redis"
```

```shell
127.0.0.1:6379[2]> mset k1 v1 k2 v2 k3 v3 # 批量设置key-value
OK
127.0.0.1:6379[2]> mget k1 k2 k3 # 批量获取key-value
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379[2]> msetnx k1 v1 k4 v4 # （原子性操作，一起成功，一起失败）批量设置时，若不存在则创建，存在则创建失败
(integer) 0
127.0.0.1:6379[2]> get k4 # 通过这里验证msetnx命令
(nil)
```

```shell
127.0.0.1:6379[2]> mset user:1:name redis user:1:age 7 # 批量设置对象，且字段name，age
OK
127.0.0.1:6379[2]> mget user:1:name user:1:age # 批量获得对象及属性
1) "redis"
2) "7"

# 这里的key是一个巧妙地设计：user:{id}:{field} 设置一个user:1 对象 值为json字符来保存一个对象
```

```shell
127.0.0.1:6379[2]> getset db redis # 如果不存在值，则返回nil
(nil)
127.0.0.1:6379[2]> get db
"redis"
127.0.0.1:6379[2]> getset db mongodb # 如果存在值，获取原来的值，并设置新的值
"redis"
127.0.0.1:6379[2]> get db
"mongodb"

# getset 先get然后set
```

String类型的使用场景：value除了是我们的字符串还可以是数字。

- 计数器
- 统计多单位地数量
- 粉丝数
- 对象缓存存储
  

## List

在Redis里面，能将List当成栈、队列、阻塞队列

所有的List命令都是以l开头的。

```shell
127.0.0.1:6379[2]> lpush list l1 # 向list添加元素，从左边添加
(integer) 1
127.0.0.1:6379[2]> lpush list l2
(integer) 2
127.0.0.1:6379[2]> lpush list l3
(integer) 3
127.0.0.1:6379[2]> lrange list 0 -1 # 获取指定区间的list元素
1) "l3"
2) "l2"
3) "l1"
127.0.0.1:6379[2]> lrange list 0 1 
1) "l3"
2) "l2"
127.0.0.1:6379[2]> rpush list right # 将元素从右边添加进去
(integer) 4
127.0.0.1:6379[2]> lrange list 0 -1
1) "l3"
2) "l2"
3) "l1"
4) "right"
```

```shell
127.0.0.1:6379[2]> lrange list 0 -1
1) "l3"
2) "l2"
3) "l1"
4) "right"
127.0.0.1:6379[2]> lpop list # 从左边移除list中的第一个值
"l3"
127.0.0.1:6379[2]> rpop list # 从右边移除list的第一个值
"right"
127.0.0.1:6379[2]> lrange list 0 -1
1) "l2"
2) "l1"
```

```shell
127.0.0.1:6379[2]> lrange list 0 -1
1) "l2"
2) "l1"
127.0.0.1:6379[2]> lindex list 1 # 获取下标 1 的值
"l1"
127.0.0.1:6379[2]> lindex list 0 # 获取下标 0 的值
"l2"

# lindex 获取指定下标的值 下标从0开始
```

```shell
127.0.0.1:6379[2]> lpush list l1 l2 l3
(integer) 3
127.0.0.1:6379[2]> llen list # 获取指定list的长度
(integer) 3

```

```shell
127.0.0.1:6379[2]> lrange list 0 -1
1) "l3"
2) "l3"
3) "l2"
4) "l1"
127.0.0.1:6379[2]> lrem list 1 l3 # 移除list中指定的值 第三个参数是count
(integer) 1
127.0.0.1:6379[2]> lrange list 0 -1
1) "l3"
2) "l2"
3) "l1"
127.0.0.1:6379[2]> lpush list l2
(integer) 4
127.0.0.1:6379[2]> lrem list 2 l2
(integer) 2
127.0.0.1:6379[2]> lrange list 0 -1
1) "l3"
2) "l1"
```

```shell
127.0.0.1:6379[2]> rpush list 1 2 3 4
(integer) 4
127.0.0.1:6379[2]> lrange list 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
127.0.0.1:6379[2]> ltrim list 1 2 # 截取下标为1，2的list元素，但此时list已经被改变！
OK
127.0.0.1:6379[2]> lrange list 0 -1
1) "2"
2) "3"

# ltrim 截取指定长度的list
```

```shell
127.0.0.1:6379[2]> lrange list 0 -1
1) "2"
2) "3"
3) "4"
127.0.0.1:6379[2]> rpoplpush list list1 # 将list最右边的值弹出从左边添加到list1
"4"
127.0.0.1:6379[2]> lrange list 0 -1
1) "2"
2) "3"
127.0.0.1:6379[2]> lrange list1 0 -1
1) "4"

# rpoplpush 将右边的值弹出从左边添加到新列表
```

```shell
127.0.0.1:6379[2]> lrange list 0 -1
1) "2"
2) "3"
127.0.0.1:6379[2]> lset list 0 22 # 将list下标为0的位置的元素设置为22
OK
127.0.0.1:6379[2]> lrange list 0 -1
1) "22"
2) "3"
127.0.0.1:6379[2]> lset list 2 4 # 若下标超出数组则Error
(error) ERR index out of range

```

```shell
127.0.0.1:6379[2]> lrange list 0 -1
1) "22"
2) "3"
127.0.0.1:6379[2]> linsert list before "3" "4" # 在3插入4
(integer) 3
127.0.0.1:6379[2]> lrange list 0 -1
1) "22"
2) "4"
3) "3"
127.0.0.1:6379[2]> linsert list after "3" "7" # 在3后面插入7
(integer) 4
127.0.0.1:6379[2]> lrange list 0 -1
1) "22"
2) "4"
3) "3"
4) "7"

# linsert 指定位置插入元素
```

小结：

- 实际上是一个链表，before Node after，left，right都可以插入值
- 如果key不存在，创建新的链表
- 如果key存在，新增内容
- 如果移除了所以值，空链表，代表不存在。
- 在两边插入或者改动值效率最高。中间元素相对来说效率低一点。
  

## set

set中的值是不能重复的。

```shell
127.0.0.1:6379[2]> sadd set 1 2 3 4 # 向set中添加元素，添加单个值则是 sadd set 1
(integer) 4
127.0.0.1:6379[2]> smembers set # 查看set中的元素
1) "1"
2) "2"
3) "3"
4) "4"
127.0.0.1:6379[2]> sismember set 2 # 查看set中是否含有2，若有则返回1，没有则返回0
(integer) 1
127.0.0.1:6379[2]> sismember set 5
(integer) 0
127.0.0.1:6379[2]> scard set # 获取set中的值的个数
(integer) 4
```

```shell
127.0.0.1:6379[2]> srem set 2 # 移除set中的指定元素
(integer) 1
127.0.0.1:6379[2]> scard set
(integer) 3
127.0.0.1:6379[2]> smembers set
1) "1"
2) "3"
3) "4"
```

```shell
# set无序不重复！
127.0.0.1:6379[2]> srandmember set # 随机取出一个元素
"4"
127.0.0.1:6379[2]> srandmember set 2 # 随机取出2个元素
1) "4"
2) "1"
127.0.0.1:6379[2]> srandmember set 2
1) "3"
2) "4"
```

```shell
127.0.0.1:6379[2]> smembers set
1) "1"
2) "3"
3) "4"
127.0.0.1:6379[2]> spop set # 随机删除一个元素
"4"
127.0.0.1:6379[2]> smembers set
1) "1"
2) "3
```

```shell
127.0.0.1:6379[2]> smove set set1 3 # 将set中的元素3移动到set1中
(integer) 1
127.0.0.1:6379[2]> smembers set
1) "1"
127.0.0.1:6379[2]> smembers set1
1) "3"
2) "6"
3) "7"
4) "8"
```

```shell
127.0.0.1:6379[2]> sadd set1 a b c
(integer) 3
127.0.0.1:6379[2]> sadd set2 c d e
(integer) 3
127.0.0.1:6379[2]> sdiff set1 set2 # 查看set1和set2中不相同的元素
1) "b"
2) "a"
127.0.0.1:6379[2]> sinter set1 set2 # 查看set1和set2中相同的元素
1) "c"
127.0.0.1:6379[2]> sunion set1 set2 # 查看set1和set2的并集
1) "e"
2) "a"
3) "c"
4) "b"
5) "d"
```



## Hash

Map集合，key-Map。这时候的key-value中的value是map集合。本质和String没有太大区别

```shell
127.0.0.1:6379[2]> hset hash f1 hello # 通过hset创建hash，key-value为 f1-hello
(integer) 1
127.0.0.1:6379[2]> hget hash f1 # 获取指定字段值
"hello"
127.0.0.1:6379[2]> hmset hash f2 haha f3 ahah f4 xixi # 批量设置key-value
OK
127.0.0.1:6379[2]> hmget hash f1 f2 f3 f4 # 批量获取key-value
1) "hello"
2) "haha"
3) "ahah"
4) "xixi"
127.0.0.1:6379[2]> hgetall hash # 获取hash里的所有键值
1) "f1"
2) "hello"
3) "f2"
4) "haha"
5) "f3"
6) "ahah"
7) "f4"
8) "xixi"
```

```shell
127.0.0.1:6379[2]> hdel hash f1 f2 # 删除hash中的指定字段
(integer) 2
127.0.0.1:6379[2]> hgetall hash
1) "f3"
2) "ahah"
3) "f4"
4) "xixi"
```

```shell
127.0.0.1:6379[2]> hset hash f5 aaa f6 bbb
(integer) 2
127.0.0.1:6379[2]> hgetall hash
1) "f3"
2) "ahah"
3) "f4"
4) "xixi"
5) "f5"
6) "aaa"
7) "f6"
8) "bbb"
127.0.0.1:6379[2]> hlen hash # 获取hash的长度
(integer) 4
```

```shell
127.0.0.1:6379[2]> hexists hash f2
(integer) 0
127.0.0.1:6379[2]> hexists hash f3
(integer) 1

# 判断hash表指定字段是否存在
```

```shell
127.0.0.1:6379[2]> hkeys hash # 获取所有key
1) "f3"
2) "f4"
3) "f5"
4) "f6"
127.0.0.1:6379[2]> hvals hash # 获取所有value
1) "ahah"
2) "xixi"
3) "aaa"
4) "bbb"
```

```shell
127.0.0.1:6379[2]> hsetnx hash f1 6 # 若字段f1不存在则创建，存在则创建失败
(integer) 1
127.0.0.1:6379[2]> hsetnx hash f1 8
(integer) 0
127.0.0.1:6379[2]> hincrby hash f1 1 # 将f1字段的值增加步长1
(integer) 7
127.0.0.1:6379[2]> hget hash f1 # 获取f1字段的值
"7"
```



## Zset（有序集合）

```shell
127.0.0.1:6379[2]> zadd zset 1 one 2 two 3 three # 向zset中添加多个值
(integer) 3
127.0.0.1:6379[2]> zrange zset 0 -1 # 取出所有值
1) "one"
2) "two"
3) "three"
```

```shell
127.0.0.1:6379[2]> zadd salary 10000 xh
(integer) 1
127.0.0.1:6379[2]> zadd salary 8000 xm
(integer) 1
127.0.0.1:6379[2]> zadd salary 7000 xa
(integer) 1
127.0.0.1:6379[2]> zrange zset 0 -1
1) "one"
2) "two"
3) "three"
127.0.0.1:6379[2]> zrange salary 0 -1
1) "xa"
2) "xm"
3) "xh"
127.0.0.1:6379[2]> zrangebyscore salary -inf +inf withscores # zrangebyscore 排序，从小打到（负无穷到正无穷方向）
1) "xa"
2) "7000"
3) "xm"
4) "8000"
5) "xh"
6) "10000"
127.0.0.1:6379[2]> zrangebyscore salary -inf 8000 withscores
1) "xa"
2) "7000"
3) "xm"
4) "8000"
127.0.0.1:6379[2]> zrevrange salary 0 -1 # 倒序查看
1) "xh"
2) "xm"
```

```shell
127.0.0.1:6379[2]> zrange salary 0 -1
1) "xa"
2) "xm"
3) "xh"
127.0.0.1:6379[2]> zrem salary xa # 移除指定元素
(integer) 1
127.0.0.1:6379[2]> zrange salary 0 -1
1) "xm"
2) "xh"
127.0.0.1:6379[2]> zcard salary # 获取集合中的元素个数
(integer) 2
```

```shell
127.0.0.1:6379[2]> zadd set1 1 hello 2 world 3 aabb
(integer) 3
127.0.0.1:6379[2]> zcount set1 0 2 # 查看某个区间内的元素个数，这里注意 我写的0 2，表示0，1，2 我添加的时候索引是1，2，3 所以在这个区间内的元素个数有2个
(integer) 2
127.0.0.1:6379[2]> zcount set1 1 3
(integer) 3
```



# 三种特殊数据类型



## geospatial 地理位置

定位、附近的人、打车距离计算

```shell
127.0.0.1:6379[2]> geoadd china:city 104.06 30.65 chengdu 105.44 28.88 luzhou 106.50 29.53 chongqing 114.08 22.54 shenzhen 121.47 31.23 shanghai
(integer) 5 
# 通过geoadd 命令添加城市地理位置，但是两级不能添加 参数：key 纬度 经度 value
# 不能超过有效经度纬度范围：
#   Valid longitudes are from -180 to 180 degrees.
#   Valid latitudes are from -85.05112878 to 85.05112878 degrees.

127.0.0.1:6379[2]> geopos china:city chengdu luzhou # 获取指定城市的经度纬度
1) 1) "104.05999749898910522"
   2) "30.6499990746355806"
2) 1) "105.43999940156936646"
   2) "28.88000075434802483"
```

```shell
# geodist 城市之间的直线距离
# 单位
#   m for meters.
#   km for kilometers.
#   mi for miles.
#   ft for feet.
127.0.0.1:6379[2]> geodist china:city chengdu luzhou
"237714.9128"
127.0.0.1:6379[2]> geodist china:city chengdu luzhou km # km表示单位为千米
"237.7149"
```

```shell
# georadius 以指定经度纬度为原点，指定距离为半径，查找周围城市
127.0.0.1:6379[2]> georadius china:city 100 30 1000 km withcoord # 以100，30为中心，寻找方圆1000km以内的城市
1) 1) "luzhou"
   2) 1) "105.43999940156936646"
      2) "28.88000075434802483"
2) 1) "chengdu"
   2) 1) "104.05999749898910522"
      2) "30.6499990746355806"
3) 1) "chongqing"
   2) 1) "106.49999767541885376"
      2) "29.52999957900659211"
127.0.0.1:6379[2]> georadius china:city 100 30 1000 km withcoord count 1 # 显示经纬度信息
1) 1) "chengdu"
   2) 1) "104.05999749898910522"
      2) "30.6499990746355806"
127.0.0.1:6379[2]> georadius china:city 100 30 1000 km withdist # 显示直线距离
1) 1) "luzhou"
   2) "541.4012"
2) 1) "chengdu"
   2) "396.4148"
3) 1) "chongqing"
   2) "629.6756"
```

```shell
# georadiusbymember 通过城市名为中心，朝朝周围的城市
127.0.0.1:6379[2]> georadiusbymember china:city chengdu 500 km
1) "luzhou"
2) "chengdu"
3) "chongqing"
127.0.0.1:6379[2]> georadiusbymember china:city chengdu 500 km withcoord # 携带经纬度
1) 1) "luzhou"
   2) 1) "105.43999940156936646"
      2) "28.88000075434802483"
2) 1) "chengdu"
   2) 1) "104.05999749898910522"
      2) "30.6499990746355806"
3) 1) "chongqing"
   2) 1) "106.49999767541885376"
      2) "29.52999957900659211"
```

```shell
# geohash 返回城市的位置hash串
127.0.0.1:6379[2]> geohash china:city chengdu luzhou
1) "wm3yrgwjjt0"
2) "wm4ur37n4n0"
```

geo底层的实现原理就是Zset！我们可以使用Zset命令操作geo

```shell
127.0.0.1:6379[2]> zrange china:city 0 -1
1) "luzhou"
2) "chengdu"
3) "chongqing"
4) "shenzhen"
5) "shanghai"
127.0.0.1:6379[2]> zrem china:city shanghai
(integer) 1
127.0.0.1:6379[2]> zrange china:city 0 -1
1) "luzhou"
2) "chengdu"
3) "chongqing"
4) "shenzhen"
```



## Hyperloglog

什么是基数？不重复的元素，接收误差

简介：基数统计的算法。

优点：占用内存是固定的，2^64不同的元素，只需要12KB内存！

```shell
127.0.0.1:6379[2]> PFadd mykey a b c d e f g h i j # 使用pfadd添加
(integer) 1
127.0.0.1:6379[2]> PFcount mykey # pfcount 计算总数
(integer) 10
127.0.0.1:6379[2]> PFadd mykey2 i j k s f v e w o 
(integer) 1
127.0.0.1:6379[2]> PFcount mykey2
(integer) 9
127.0.0.1:6379[2]> PFmerge mykey3 mykey mykey2 # pfmerge合并
OK
127.0.0.1:6379[2]> PFcount mykey3
(integer) 15
```



## Bitmaps

位存储

统计用户信息，活跃，不活跃。登录、未登录...

使用二进制来记录

示例：记录周一到周日的打卡记录，0代表打卡，反之。

```shell
127.0.0.1:6379[2]> setbit sign 0 1 # setbit设置
(integer) 0
127.0.0.1:6379[2]> setbit sign 1 1 
(integer) 0
127.0.0.1:6379[2]> setbit sign 2 1
(integer) 0
127.0.0.1:6379[2]> setbit sign 3 0
(integer) 0
127.0.0.1:6379[2]> setbit sign 4 0
(integer) 0
127.0.0.1:6379[2]> setbit sign 5 1
(integer) 0
127.0.0.1:6379[2]> setbit sign 6 0
(integer) 0
127.0.0.1:6379[2]> getbit sign 6 # geibit获取
(integer) 0
127.0.0.1:6379[2]> bitcount sign # 统计打卡天数
(integer) 4
```
