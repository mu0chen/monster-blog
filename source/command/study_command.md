---
title: Study Command
date: 2025-08-06 10:48:20
---



# 0. 引言

本文的目的是收集一些常用的命令，并对这些命令做出相关详细的解释。



## 1. Kafka命令

查看当前Kafka的版本：

```shell
kafka-topic.sh --version
```

查看Kafka中的所有topic

```shell
# 若kafka version <= 2.7
kafka-topic.sh --zookeeper localhost:2181 --list
# 若kafka version > 2.7
kafka-topic.sh --bootstrap-server localhost:9092 --list
```

有这个区别的原因是，早期的kafka中，zookeeper作为kafka集群的核心依赖，用于管理集群的元数据，比如集群borker列表、Topic的分区信息及其副本的分配、控制选举、消费者组的偏移量等等，而为了减少多zookeeper的依赖，提高安全性，就开始引入bootstrap-server，它可以直接和broker通信，从broker获取整个集群的信息。

向某个kafka队列中打入数据

```shell
kafka-console-producer.sh --broker-list localhost:9092 --topic {topic_name}
```





## 2. Postgresql命令

查看当前会话语句的超时时间&临时修改（时常运行一些长消耗语句时，需要对其进行临时修改）

```sql
show statement_timeout;
set statement_timeout = 3600;      --注意单位是ms
```

查看当前语句运行的执行计划以及执行所消耗的时间（真实执行计划）

```sql
explain analyze {sql statement} --列出查询计划以及时间
explain (analyze, buffers) {sql statement} --列出查询计划以及时间以及buffer的命中情况
```





## 3. Redis命令

连接redis组件

```shell
redis-cli -p 6379 -a {password}
```

选择某个redis数据库

```shell
select 0;        -- 选择0号数据库
```





## 4. Clickhouse命令

连接Clickhouse

```shell
clickhouse-client -u {username} --port=9100 --password={password}
```





## 5. Python误区

```python
# module_a.py
logger =  None

# module_b.py
from module_a import logger
logger = Logger()

# module_c.py
from module_a import logger
print(logger)          --None
```

当使用import导入另外一个py文件中的全局变量时，将会绑定到当前模块的命名空间里中，当然，这样的变量同样也需要分可变类型和不可变类型，其中如果导入不可变类型时，例如整数、字符串、元祖，这种导入，实质上是创建了一个新的对象，原始模块中的变量将不受影响，但是







## 6. ElasticSearch命令

通常可以通过HTTP的方式与elasticsearch进行交互，比较常见的就是使用curl命令进行

查看es的所有索引：

```shell
curl /_cat/indices?v
```





