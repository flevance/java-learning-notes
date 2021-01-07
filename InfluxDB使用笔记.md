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



## 注意事项

### 重复数据

一个点由measurement名称，tag set和timestamp唯一标识。如果有相同measurement，tag set和timestamp，但具有不同field set的行协议，则field set将变为旧field set与新field set的合并，并且如果有任何冲突以新field set为准。

