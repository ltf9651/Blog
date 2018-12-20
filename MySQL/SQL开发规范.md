## SQL开发规范
- 使用预编译语句
  - 可防止sql注入
  - 减少编译时间
  - 提高效率
- 避免数据类型的隐式转换
```
SELECT name from table where id = '1' // id(INT) '1'(String) 索引失效
```
- 充分利用已存在的索引
  - 避免使用双`%`的查询
  - 一个sql语句只利用到复合索引中的一列进行查询
  - 使用`not exists`代替`not in`
- 禁止进行跨库查询
- 禁止使用`SELECT (*)`  
  - 消耗系统资源
  - 无法使用覆盖索引
  - 可减少对表结构变更带来的影响
- Insert语句禁止省略字段
```
INSERT INTO table VALUES('1', '2', '3') // wrong
INSERT INTO table(id1, id2, id3) VALUES('1', '2', '3') // right
```
- 避免使用子查询，可将子查询优化为`join`操作
  - 子查询返回的结果集无法使用索引，性能差
  - 子查询产生临时表，消耗系统资源，数据量大时影响操作
- 避免`join`太多表
  - 每`join`一个表就多占用一部分内存
  - 产生临时表
  - `join`操作理论上最多可以关联 **61** 张表，建议不超过 **5** 个
- 减少与数据库的交互次数
  - 数据库更适合处理批量操作
  ```
  SELECT name FROM table LIMIT 100
  SELECT name FROM table LIMIT 1
  //这两条sql语句效率基本一致
  ```
- 使用`in`代替`or`
  - `in`可以更高效的利用索引
- 禁止使用`order by rand()`进行随机排序
- 在`where`从句中禁止对列进行函数转换和计算
```
SELECT name FROM table WHERE date(time) = '20180101' //此查询无法使用索引
SELECT name FROM table WHERE time >= '20180101' AND time <= '20180101' //优化
```
- 在明显不会有重复值的时候使用`Union All`代替`Union`
  - `Union All`会把数据放到临时表中进行去重
- 拆分大sql语句为多个小sql
  - 拆分后通过并行执行提高效率