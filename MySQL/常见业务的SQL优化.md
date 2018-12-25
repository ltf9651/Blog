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