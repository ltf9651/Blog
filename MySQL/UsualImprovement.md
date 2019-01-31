## 常见业务的SQL优化

1. 大数据量分页
    ```sql
    SELECT
	comment_id,
	title,
	content TABLE
    WHERE
        id = 1
    AND state = 1
    AND p_id = 187985
    LIMIT 0,
    10;
    ```

    优化方式①：使用联合索引index(p_id, state) (区分度：p_id > state)
    ```sql
    SELECT
	t.comment_id,
	t.title,
	t.content
    FROM
        (
            SELECT
                'comment_id'
            FROM
                TABLE
            WHERE
                p_id = 187985
            AND state = 1
            LIMIT 0,
            5
        ) a
    JOIN TABLE t ON a.comment_id = t.comment_id
    // 先做一个子查询查出 id（只会在索引里面扫描），然后关联查询，这样扫描的行数是限定的。而不会扫描表前面所有的行。
    ```

    优化方式②：分段扫描，循环翻页的时候会记录下次查询的起始id
    ```sql
    SELECT
	comment_id,
	title,
	content
    FROM
        TABLE
    WHERE
        id > 187985
    LIMIT 0,
    5;
    ```

1. 删除重复数据
    ```sql
    CREATE TABLE bak_table_20180101 AS SELECT
	*
    FROM
        TABLE (LIKE TABLE);
    // 首先进行备份，数据量大使用mysqldump
    ```

    ```sql
    // 删除大于最小c_id的数据(保留最早的comment)
    DELETE a
    FROM
        TABLE a
    JOIN (
        SELECT
            order_id,
            p_id,
            MIN(comment_id) AS c_id
        FROM
            TABLE
        GROUP BY
            order_id,
            p_id
        HAVING
            count(*) >= 2
    ) b ON a.order_id = b.order_id
    AND a.p_id = b.p_id
    AND a.comment_id > b.c_id
    ```

1. 分区间统计
    - 先找order表， group by customer_id as b
    - 使用 case when 进行分区键 as a
    - a left join b on a.customer_id = b.customer_id

1. 优化 `not in` ， '>'， '<'
    ```sql
    // 查询所有没有产生过消费的用户信息
    SELECT
	customer_id,
	NAME,
	email
    FROM
        customer
    WHERE
        customer_id NOT IN (
            SELECT
                customer_id
            FROM
                payment
        ) // 对payment表多次读取

    // 优化
    SELECT
	a.customer_id,
	a. NAME,
	a.email
    FROM
        customer a
    LEFT JOIN payment b ON a.customer_id = b.customer_id
    WHERE
        b.customer_id IS NULL
    ```

1. 使用汇总表优化查询
    ```sql
   SELECT
	count(*)
    FROM
        COMMENT
    WHERE
        product_id = 100;

    // 优化
    CREATE TABLE comment_cnt (product_id INT, cnt INT);

    SELECT
        SUM(cnt)
    FROM
        (
            SELECT
                cnt
            FROM
                comment_cnt
            WHERE
                product_id = 100
            UNION ALL
                SELECT
                    COUNT(*)
                FROM
                    COMMENT
                WHERE
                    product_id = 100
                AND time > DATE(NOW())
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