## 索引优化

使用索引的好处
1. 减少数据扫描量
1. 避免临时表，提高排序效率
1. 化随机IO为顺序IO

索引分类
1. Btree索引：适用于`=, <, >, like, >=, <=`等
1. Hash索引：只适用于`=`

效率上` Hash > Btree `

### BTree索引
适用场景
1. 全值匹配的查询
1. 匹配最左前缀的查询
1. 匹配列前缀的查询：`id like '123%`
1. 匹配范围值：`id > 1992 and id < 2322`
1. 匹配精确左前列并范围匹配另一列：`id = 123 and price > 99`
1. 只访问索引的查询：覆盖索引
1. Order by排序

限制
1. 不按照索引最左列开始查找（联合索引乱序)
1. 不能跳过联合索引中左边的列
1. `not in`、`<`、`>`无法使用索引
1. 如果查询中有某个列的范围查询，其右边的所有列都无法使用索引

**Btree索引可以加快查询速度，更适合范围查找**

### Hash索引
基于Hash表实现，只有查询条件**精确匹配**Hash索引中所有列的时候才可以使用到

限制
1. 由于Hash全值匹配，对读改频繁的系统不建议使用
1. 无法用于排序
1. 不支持部分索引查找和范围查找
1. 可能存在Hash冲突
1. 不支持覆盖索引
1. 查询列数过多效率下降


### 优化策略
1. 对索引列不适用函数或表达式
    ```sql
    WHERE to_days(date) - to_days(date) <= 30

    // 优化
    WHERE date < date_add(current_date, interval 30 day)
    ```

1. 使用覆盖索引
    - 可减少磁盘IO，优化缓存（只读取索引)
    - 可减少随机IO，化随机IO为顺序IO
    - 可避免对Innodb的主键索引的二次查询
    - 可避免对MyISAM表进行系统调用

1. 使用索引扫描优化排序的条件
    1. 索引的顺序和order by 子句顺序一致
    1. 索引中的所有列方向（升序、降序）和order by 子句一致
    1. order by 的字段全部在关联表的第一张表中

1. 对长字符串使用Hash索引（使用Btree索引模拟Hash索引)
    ```sql
    ALTER INDEX idx_md5 ON TABLE (title_md5);

    SELECT
        id
    FROM
        TABLE
    WHERE
        title_md5 = md5('string')
    AND title = 'string';
    //对title_md5, title同时查询可以避免Hash冲突

    //适用于处理键值的全局匹配，所使用的Hash函数决定索引键的大小
    ```

1. 利用索引优化锁
    - 可减少锁定的行数
    - 可加快处理速度和锁的释放

1. 删除冗余、重复的索引
    - `primary key(id)`, `index(id)`, `unique id(id)`
    - `primary key(id)`, `index(a, id)`
    - `index(a)`, `index(a, b)`
    - `pt-duplicate-key-checker`检查冗余、未被使用过的索引

1. 索引的更新与维护
    - `analyze table table_name` 更新索引统计信息，减少索引碎片
    - `optimize table table_name` 维护索引碎片，会所表