### 生命周期

#### CLI模式

`php_module_startup` 模块初始化 ->> `php_request_startup` 请求初始化 ->> `php_execute_script` 脚本执行 ->> `php_request_shutdown` 请求关闭 ->> `php_module_shutdown` 模块关闭

##### php_module_startup

``` C
struct _sapi_module_struct {}
```

#### FPM模式

