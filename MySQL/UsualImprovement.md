## 常见业务的SQL优化

1. 大数据量分页
    ```
    select comment_id, title, content table where id = 1 and state = 1 and p_id = 187985 limit 0, 10;
    ```

    优化方式①：使用联合索引index(p_id, state) (区分度：p_id > state)
    ```
    select t.comment_id, t.title, t.content from
        ( select 'comment_id' from table
        where p_id = 187985 and state = 1 limit 0, 5 ) a
    join table t on a.comment_id = t.comment_id
    ```

    优化方式②：循环扫描，记录下次查询的起始id
    ```
    select comment_id, title, content from table where id > 187985 limit 0, 5;
    ```

1. 删除重复数据
    ```
    create table bak_table_20180101 as select * from table(like table); // 首先进行备份，数据量大使用mysqldump
    ```

    ```
    删除大于最小c_id的数据(保留最早的comment)
    delete a
    from table a
    join(
        select order_id, p_id, MIN(comment_id) as c_id
        from table
        group by order_id, p_id
        having count(*) >= 2
    ) b
    on a.order_id = b.order_id and a.p_id = b.p_id and a.comment_id > b.c_id
    ```

1. 分区间统计
    - 先找order表， group by customer_id as b
    - 使用 case when 进行分区键 as a
    - a left join b on a.customer_id = b.customer_id

1. 优化 `not in` ， '>'， '<'
    ```
    // 查询所有没有产生过消费的用户信息
    select customer_id, name, email from customer where customer_id NOT IN (
        select customer_id from payment
    ) // 对payment表多次读取

    // 优化
    select a.customer_id, a.name, a.email from customer a 
    LEFT JOIN
    payment b
    ON a.customer_id = b.customer_id
    WHERE b.customer_id IS NULL
    ```

1. 使用汇总表优化查询
    ```
    select count(*) from comment where product_id = 100;

    // 优化
    create table comment_cnt(product_id INT, cnt INT);
    select SUM(cnt) from (
        select cnt from comment_cnt where product_id = 100
        UNION ALL
        select COUNT(*) from comment where product_id = 100 and time > DATE(NOW())
    ) a 
    ```

1. 千万计表的优化
    - 表非存储在连续的物理空间上，而是由链式存储在多个碎片的物理空间上
    - 优化方式1：对表拆分，减少字段数，优化表结构
      - 水平拆分：RANGE、HASH取模，具有IO瓶颈
      - 垂直拆分：表拆分后放于多个服务器上
        - ID 1-10000 `select,limit,order by`
        - ID 10001-20000 `select,limit,order by`
        - ID 20001-30000 `select,limit,order by`
        - 将上述三条结果集合并，再次排序
    - 优化方式2：调整字段顺序与索引顺序一致

1. `in`和`exists`
    - `in` 适用于小表
    - `exists` 适用于大表
    - `not exists` > `not in`，无论表的大小（`not in`会进行全表扫描，无法使用索引)