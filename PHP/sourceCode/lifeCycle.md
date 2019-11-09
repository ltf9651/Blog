### 生命周期

#### CLI模式

`php_module_startup` 模块初始化 ->> `php_request_startup` 请求初始化 ->> `php_execute_script` 脚本执行 ->> `php_request_shutdown` 请求关闭 ->> `php_module_shutdown` 模块关闭

##### php_request_startup

1. php_output_activate() // 重置输出全局并设置输出处理程序的堆栈

#### FPM模式

* 运行模式
    - pm = static  保持固定数量的子进程数(max_children)
    - pm = dynamic  动态模式，子进程数量变化
    - pm = ondemand  按需模式，闲置进程在设定时间内会被杀死，降低低峰期的内存使用，但高峰期会频繁创建进程

