## 网络编程

### Server 接收步骤

1. socket 创建fd
1. bind 设置端口号，进行绑定
1. listen 开始监听
1. accept 接收请求

### FPM 对信号的处理

* KILL SIGUSER1 php-fpm 重新打开日志文件
* KILL SIGUSER2 php-fpm 重载配置，平滑重启所有worker进程

* Master进程负责管理、分配、监听worker进程，不进行服务，kill Master进程后，worker仍可以进行服务
* kill worker进程后，Master进程会新起另一个worker进程进行服务
* worker进程对服务进行争抢，抢到的进行服务

* 504 处理时间过长
* 502 连不上fpm

