## SQL开发规范
- 使用预编译语句
  - 避免动态sql带来的sql注入
  - 减少编译时间
  - 高效，节省带宽
  - 一次解析，多次使用
- 避免数据类型的隐式转换
```sql
SELECT name from table where id = '1' // id(INT) '1'(String) 索引失效
```
- 充分利用已存在的索引
  - 避免使用双`%`、前`%`的查询（索引失效），使用后`%`可利用索引
  - 一个sql语句只利用到复合索引中的一列进行查询
  - 使用`not exists`代替`not in`(索引失效)
  - `index(a, b, c)`->若对a进行范围查询频繁->`index(b, c, a)`
- 程序连接不同数据库时使用不同账号，禁止进行跨库查询
  - 为库迁移、分库分表留出余地
  - 降低业务耦合度
  - 避免权限过大产生的风险
- 禁止使用`SELECT (*)`
  - 消耗系统资源
  - 无法使用覆盖索引
  - 可减少对表结构变更带来的影响
- Insert语句禁止省略字段
```sql
INSERT INTO table VALUES('1', '2', '3') // wrong
INSERT INTO table(id1, id2, id3) VALUES('1', '2', '3') // right
```
- 避免使用子查询，可将子查询优化为`join`操作
  - 子查询返回的结果集无法使用索引，性能差
  - 子查询返回的结果存储在临时表，无法使用索引
  - 消耗系统资源，数据量大时影响操作，产生慢查询
- 避免`join`太多表
  - 每`join`一个表就多占用一部分内存（join_buffer_size）
  - 产生临时表
  - `join`操作理论上最多可以关联 **61** 张表，建议不超过 **5** 个
- 减少与数据库的交互次数
  - 数据库更适合处理批量操作
  ```sql
  SELECT name FROM table LIMIT 100
  SELECT name FROM table LIMIT 1
  // 一次取100条进行分页 > 100次取一条
  ```
- 使用`in`代替`or`
  - `in`可以更高效的利用索引
- 禁止使用`order by rand()`进行随机排序
  - 此行为会把表中所有符合条件的数据装载到内存中进行排序
  - 消耗CPU、IO、内存
  - 将随机排序在程序代码中执行
  - 直接使用`where id = x` 的时候要考虑此条数据被删除的情况
- 在`where`从句中禁止对列进行函数转换和计算
```sql
SELECT name FROM table WHERE date(time) = '20180101' //此查询无法使用索引
SELECT name FROM table WHERE time >= '20180101' AND time < '20180102' //优化
```
- 在明显不会有重复值的结果集中使用`Union All`代替`Union`
  - `UNION`将所有数据放入临时表中再去重（消耗系统资源）
  - `Union All`不进行去重
- 拆分大sql语句为多个小sql
  - 拆分后通过并行执行提高效率（一个SQL只能使用一个CPU)
- `count(*)`效率大于`count(列)`