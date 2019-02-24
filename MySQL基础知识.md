# MySQL基础知识笔记
## 创建表
* 数据类型

|数据类型|大小(字节)|用途|格式|
|:------|:--------:|:--:|:--:|
|INT|4|整数||
|FLOAT|4|单精度浮点数||
|DOUBLE|8|双精度浮点数||
|ENUM||单选，比如性别|ENUM('男','女')|
|SET||多选|SET('A','B','C')|
|DATE|3|日期|YYYY-MM-DD|
|TIME|3|时间点或持续时间|HH:MM:SS|
|YEAR|1|年份值|YYYY|
|CHAR|0~255|定长字符串||
|VARCHAR|0~255|变长字符串||
|TEXT|0~65535|长文本数据||
* CHAR 和 VARCHAR 的区别: CHAR 的长度是固定的，而 VARCHAR 的长度是可以变化的，比如，存储字符串 “abc"，对于 CHAR(10)，表示存储的字符将占 10 个字节(包括 7 个空字符)，而同样的 VARCHAR(12) 则只占用4个字节的长度，增加一个额外字节来存储字符串本身的长度，12 只是最大值，当你存储的字符小于 12 时，按实际长度存储。
## 查询
* 关键字`LIKE`在SQL语句中和通配符一起使用，通配符代表未知字符。SQL中的通配符是`_`和`%`。其中`_`代表一个未指定字符，`%`代表不定个未指定字符。
## 修改
* `alter table 表名 rename 新表名`重命名一张表
* `alter table 表名 add 列名 数据类型 约束`增加一列
* `alter table 表名 drop 列名` 删除一列
* `alter table 表名 change 原列名 新列名 数据类型 约束`重定义一列
* `alter table 表名 modify 列名 新数据类型`改变某列数据类型
## 其他基本操作
* `create index 索引名 on 表名(列名)`建立索引
* `create view 视图名(列a,列b,列c) as select 列1,列2,列3 from 表名`创建视图
* `select 列1，列2 into outfile '文件路径和文件名' from 表名字`导出表数据
* `load data infile '文件路径和文件名' into table 表名`导入表数据
* `mysqldump -u root 数据库名 > 备份文件名`备份数据库
* `source 文件路径和文件名`或者`mysql -u root 数据库名 < 文件路径和文件名`恢复数据库
* 使用用户变量，例子：`SELECT @min_price:=MIN(price),@max_price:=MAX(price) FROM shop;`
## 触发器
* `create trigger 触发器名字 触发时间(before|after) 触发事件(insert|update|delete) on 表名 for each row 触发后操作`
* 同一表同一触发时间、事件的触发器只能有一个。
* 在`BEFORE`触发器中，具有AUTO_INCREMENT的列的NEW值为0，而不是实际插入新记录时将自动生成的序列号。
