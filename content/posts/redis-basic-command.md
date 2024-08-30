+++
title = 'Redis基本命令'
date = 2023-08-29T12:03:50+08:00
draft = false
categories = ["数据库", "教程", "redis教程"]
tags = ["Redis", "NoSQL", "缓存"]
+++
Redis提供了丰富的命令集来操作其数据结构。本文将介绍键（Key）操作命令以及各数据类型的CRUD（创建、读取、更新、删除）操作。

## 键（Key）操作命令

1. **EXISTS key**：检查键是否存在
   ```
   EXISTS mykey
   ```

2. **DEL key**：删除键
   ```
   DEL mykey
   ```

3. **TYPE key**：返回键所存储的值的类型
   ```
   TYPE mykey
   ```

4. **EXPIRE key seconds**：设置键的过期时间
   ```
   EXPIRE mykey 60
   ```

5. **TTL key**：查看键剩余的过期时间
   ```
   TTL mykey
   ```

## 各数据类型的CRUD操作

### 字符串（String）

1. 创建和更新：SET
   ```
   SET mykey "Hello"
   ```

2. 读取：GET
   ```
   GET mykey
   ```

3. 删除：DEL
   ```
   DEL mykey
   ```

### 列表（List）

1. 创建和更新：LPUSH, RPUSH
   ```
   LPUSH mylist "world"
   RPUSH mylist "hello"
   ```

2. 读取：LRANGE
   ```
   LRANGE mylist 0 -1
   ```

3. 更新：LSET
   ```
   LSET mylist 0 "new world"
   ```

4. 删除：LPOP, RPOP
   ```
   LPOP mylist
   ```

### 集合（Set）

1. 创建和更新：SADD
   ```
   SADD myset "Hello"
   ```

2. 读取：SMEMBERS
   ```
   SMEMBERS myset
   ```

3. 删除：SREM
   ```
   SREM myset "Hello"
   ```

### 有序集合（Sorted Set）

1. 创建和更新：ZADD
   ```
   ZADD myzset 1 "one"
   ```

2. 读取：ZRANGE
   ```
   ZRANGE myzset 0 -1 WITHSCORES
   ```

3. 更新：ZADD（使用相同的成员会更新分数）
   ```
   ZADD myzset 2 "one"
   ```

4. 删除：ZREM
   ```
   ZREM myzset "one"
   ```

### 哈希（Hash）

1. 创建和更新：HSET
   ```
   HSET user:1000 name "John Doe" age 30
   ```

2. 读取：HGET, HGETALL
   ```
   HGET user:1000 name
   HGETALL user:1000
   ```

3. 更新：HSET
   ```
   HSET user:1000 age 31
   ```

4. 删除：HDEL
   ```
   HDEL user:1000 age
   ```

这些是Redis中最基本和常用的命令。熟练掌握这些命令将帮助你更有效地使用Redis进行数据操作。