## 捕获有问题，需要优化的SQL

`
set global low_query_log = on; // 开启慢查询日志
`

`
set global slow_query_log_file = /sql_logs/slow_sql.log; // 慢查询日志存储路径
`

`
set global log_queries_not_using_indexes = on; // 未使用索引的sql记录
`

`
set global log_query_time = 0.5; // 查询时间大于0.5秒的记录
`


`
mysqldumpslow _slow_sql.log   //去除慢查询日志中的重复sql记录
`


`Row_sent: 0` 表示没有使用到索引
