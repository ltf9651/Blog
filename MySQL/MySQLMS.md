## MySQL主从

主从复制
1. 主库将变更写入binlog
1. 从库的IO过程读取主库binlog并存储至Relaylog中
1. 从库的SQL连接读取Relaylog

主从配置
1. 配置MySQL参数
1. 主服务器中创建用于复制的账号
1. 全库备份Master并初始化Slave（保持主从数据一致)

M-S架构存在的问题：Master挂掉后需手动切换到Slave

解决方法：使用虚拟IP连接MySQL，改主从复制为双主热备（其中只有一个主提供写服务，另一个只提供读服务）

keeplived提供VIP，提供健康监控（ARRP协议）

## MySQL双主

开启binlog同步：`log_slave_updates=1`

M主配置:
```
auto_increment_increment = 2
auto_increment_offset = 1
# 配置后自增ID： 1,3,5,7
```

M备配置：
```
auto_increment_increment = 2
auto_increment_offset = 2
# 配置后自增ID： 2,4,6,8
```

- [MYSQL复制](http://www.cnblogs.com/cchust/p/4424576.html)
- [MYSQL半同步(semi-sync)源码实现](https://www.cnblogs.com/clouddbdever/p/5603286.html)