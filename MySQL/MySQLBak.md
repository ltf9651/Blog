## 数据库备份

备份十分重要，即使有主从复制也不可以取代备份（主从复制存在时间延迟)

备份方式
  1. 逻辑备份：存储为sql语句，耗时长，会锁表
  1. 物理备份：拷贝目录，速度快，对于内存表只能辈分结构

备份量
  1. 全量备份
  1. 增量备份

备份命令：`mysqldump`

恢复命令：`source backup.sql`

指定备份时间点：
```
mysqlbinlog --start-position = 
            --stop-position = 
            > backup.sql

# 条件：
1. 指定时间点前有全量备份
2. 开启binlog日志功能
```

mysql 5.6 之后支持实时备份二进制日志

备份工具：
  1. xtrabackup : 只支持InnoDB,，不影响读写
  1. innobackupex : 支持MYISAM，会锁表