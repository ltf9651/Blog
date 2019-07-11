## 杂七杂八~

* 浮点
```php
$a = 0.7;
$b = 0.1;
var_dump(($a + $b) == 0.8); // return false
var_dump(bcadd($a, $b, 2) == 0.8); // return true
```

- 字符串比较时转换
```php
switch ("a") {
    case  0:
        $a = 1;
        # code...
        break;

    default:
        # code...
        $a = 2;
        break;
}
var_dump('a' == 0); // bool true
echo $a; // $a = 1;
```

* **Bool**
``` php
0 == 0.0 == '' == '0' == false == array() == null
```

- 常量定义
  + const：语言结构，可以定义类常量，速度更快
  + define：函数，编译前处理

```php
null-- => null
null++ => 1
true(false) ++/-- => true(false)
```

* **Static**：变量值初始化一次（保留处理后的值）

* 找不到文件时
  * **include**: 警告，仍继续执行脚本
  * **require**：致命错误

```php
$array = array('k1' => 'v1', 'k2' => 'v2');
extract($array);
echo $k1 . ' and ' . $k2; //return 'v1 and v2'
```

### IO多路复用

多个描述符I/O操作都能在一个线程内并发的交替完成，“复用”指复用同一个线程

#### select
1. 线性扫描，不断遍历内容，效率低下
1. 能够监视文件描述符的数量存在最大限制


#### epoll
1. 每当fd就绪，采用系统的回调函数直接将fd放入，效率高
1. 无最大连接限制

### Crontab
1. 单台服务器Crontab脚本太多会导致脚本中断
1. Crontab任务数量多应分配到负载均衡的服务器上