### 生命周期

#### CLI 模式

`php_module_startup` 模块初始化 ->> `php_request_startup` 请求初始化 ->> `php_execute_script` 脚本执行 ->> `php_request_shutdown` 请求关闭 ->> `php_module_shutdown` 模块关闭

- module_startup_func: 这个函数在 PHP 模块初始化阶段执行，通常情况下，此过程只会在 SAPI 启动后执行一次。这个阶段可以进行内部类的注册，如果你的扩展提供了类就可以在此函数中完成注册；除了类还可以在此函数中注册扩展定义的常量；另外，扩展可以在此阶段覆盖 PHP 编译、执行的两个函数指针：zend_compile_file、zend_execute_ex，从而可以接管 PHP 的编译、执行，opcache 的实现原理就是替换了 zend_compile_file，从而使得 PHP 编译时调用的是 opcache 自己定义的编译函数，对编译后的结果进行缓存。

- request_startup_func: 此函数在编译、执行之前回调，fpm 模式下每一个 http 请求就是一个 request，脚本执行前将首先执行这个函数。如果你的扩展需要针对每一个请求进行处理则可以设置这个函数，如：对请求进行 filter、根据请求 ip 获取所在城市、对请求 / 返回数据加解密等。

- request_shutdown_func: 这个函数比较特殊，一般很少会用到，实际它也是在请求结束之后调用的，它比 request_shutdown_func 更晚执行

- module_shutdown_func: 模块关闭阶段回调的函数，与 module_startup_func 对应，此阶段主要可以进行一些资源的清理

#### FPM 模式

* 运行模式
    - pm = static  保持固定数量的子进程数 (max_children)
    - pm = dynamic  动态模式，子进程数量变化
    - pm = ondemand  按需模式，闲置进程在设定时间内会被杀死，降低低峰期的内存使用，但高峰期会频繁创建进程

- pm 的实现就是创建一个 master 进程，在 master 进程中创建并监听 socket，然后 fork 出多个子进程，这些子进程各自 accept 请求，子进程的处理非常简单，它在启动后阻塞在 accept 上，有请求到达后开始读取请求数据，读取完成后开始处理然后再返回，在这期间是不会接收其它请求的，也就是说 fpm 的子进程同时只能响应一个请求，只有把这个请求处理完成后才会 accept 下一个请求，这一点与 nginx 的事件驱动有很大的区别，nginx 的子进程通过 epoll 管理套接字，如果一个请求数据还未发送完成则会处理下一个请求，即一个进程会同时连接多个请求，它是非阻塞的模型，只处理活跃的套接字
- fpm 的 master 进程与 worker 进程之间不会直接进行通信，master 通过共享内存获取 worker 进程的信息，比如 worker 进程当前状态、已处理请求数等，当 master 进程要杀掉一个 worker 进程时则通过发送信号的方式通知 worker 进程
- fpm 可以同时监听多个端口，每个端口对应一个 worker pool，而每个 pool 下对应多个 worker 进程，类似 nginx 中 server 概念
- fpm 通过 master fork 出 worker 进程，每一个 worker 独自处理 accept，且是阻塞模型，一个 worker 同时只能处理一个请求
