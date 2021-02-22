## schema常用语法

~~~ sql
SHOW DATABASES # 查询数据库
SHOW RETENTION POLICIES on xxx # 返回指定数据库的保留策略的列表
SHOW SERIES on xxx # 查询指定数据的series
SHOW MEASUREMENTS # 查询数据库中的表
SHOW TAG KEYS 
SHOW TAG VALUES 
SHOW FIELD KEYS 
~~~

## 数据定义和MYSQL比较

| InFluxDB         | 释义                             | MYSQL         |
| ---------------- | -------------------------------- | ------------- |
| database         | 库                               | Database      |
| measurement      |                                  | table         |
| field            | 字段                             | field         |
| tag              | 字段（必须是字符串类型）         | 带索引的field |
| series           | measurement+tag+retention policy | 联合索引      |
| timestamp        | 时间戳                           | id            |
| retention policy | InfluxDB保存数据的时间           |               |



## 常用操作

插入
~~~
insert <measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]
~~~

查询

~~~
SELECT <stuff> FROM <measurement_name> WHERE <some_conditions>
~~~



## 注意事项

### 重复数据

一个点由measurement名称，tag set和timestamp唯一标识。如果有相同measurement，tag set和timestamp，但具有不同field set的行协议，则field set将变为旧field set与新field set的合并，并且如果有任何冲突以新field set为准。

### 不存在更新和删除

因为更新和删除会非常消耗。所以influxDB不支持这种操作

### 插入的时间一定要按照时间的先后顺序

绝大多数写入都是接近当前时间戳的数据，并且数据是按时间递增的顺序添加。按时间递增的顺序添加数据明显更高效些，随机时间或时间不按升序写入点的性能要低得多

### 不要有太多的tag列

会大大更新数据库的IO。如果tag列过多，就会导致增加很多的series。导致查询速度变慢。tag的数据长度也建议尽量短



## 拓展

### RP 数据保留时间

建库是直接指定（支持w周，d天，h小时）(数据保留一年的推荐语句)

~~~
CREATE RETENTION POLICY "on_year" ON "database_name" DURATION 52w REPLICATION 1 SHARD DURATION 1d DEFAULT
~~~

插入时指定

```
 INTO "rp_name"."measurement" tag fiel time
```

* 数据保留策略的网址介绍：https://www.cnblogs.com/ilifeilong/p/12746149.html
* 一些使用注意事项：https://developer.aliyun.com/article/758923?spm=a2c6h.14164896.0.0.2d946b37xSY3X7
* 性能和调优：https://blog.csdn.net/eric_sunah/article/details/76274188?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control
* 一些查询语法注意事项：https://blog.csdn.net/lifen0908/article/details/105293839/
* GROUP BY的相关介绍：https://www.cnblogs.com/suhaha/archive/2019/10/17/11692281.html

（一个入库，一个查询）

1. 测温
2. 环流
3. 局放
4. 载流量
5. 环境监测