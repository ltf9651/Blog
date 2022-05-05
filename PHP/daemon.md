```php

常见做法：systemd / supervisor

<?php

function daemon()
{
    // [1] 创建子进程
    $pid = pcntl_fork();
    if ($pid == -1) {
        die('fork failed');
    }

    // [2] 如果是父进程，则退出
    if ($pid > 0) {
        exit(0);
    }

    ///////////////// 以下是子进程 /////////////////

    // [3] 创建一个新的会话并成为 session leader
    if ( ($sid = posix_setsid()) <= 0 ) {
        die("Set sid failed.\n");
    }

    // [4] 重设文件掩码
    umask(0);

    // [5] 改变工作目录
    // 守护进程是长时间运行的，通常只有系统关闭/重启才会退出
    // 假如从父进程继承来的工作目录是个挂载的文件系统，如果不改变工作目录，那么将会导致这个挂载的文件系统一直没法卸载
    if (chdir('/') === false) {
        die("chdir failed.\n");
    }

    // [6] 关闭标准输入输出
    fclose(STDIN);
    fclose(STDOUT);
    fclose(STDERR);

    // [7] 重定向输入输出
    global $stdin, $stdout, $stderr;
    $stdin = fopen('/dev/null', 'r');
    $stdout = fopen('/dev/null', 'wb'); // 你也可以将标准输出重定向到指定的文件，相当于是日志
    $stderr = fopen('/dev/null', 'wb'); // 同上
}

daemon();

// ... 真正的处理逻辑
```

### 守护进程步骤

1. 创建子进程，退出父进程
1. 子进程创建一个新的会话并成为 session leader
1. 重设文件掩码
1. 改变工作目录
1. 关闭标准输入输出
