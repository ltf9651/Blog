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
set global log_query_time = 0.001; // 查询时间大于1毫秒的记录
`


`
mysqldumpslow slow_sql.log   //去除慢查询日志中的重复sql记录
`

`
mysqldumpslow -s r -t 10 slow_sql.log // 删除重复数据，汇总结果，分析结果并按排序输出
`

更强大的分析工具：`pt-query-digest`

`
pt-query-digest --explain h=127.0.0.1 slow_sql.log > result.rep
`

`Row_sent: 0` 表示没有使用到索引


实时获取有性能问题的SQL
```sql
SELECT
	id,
	'user',
	'host',
	DB,
	command,
	'time',
	state,
	info
FROM
	information_schema. PROCESSLIST
WHERE
	time >= 30;
```