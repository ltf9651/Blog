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

- 性能：`array_key_exists` > `in_array`，时间复杂度 O(1) : O(n)

- 性能：`(int)` > `intval`， `mt_rand` > `rand`

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

### IO 多路复用

多个描述符 I/O 操作都能在一个线程内并发的交替完成，“复用”指复用同一个线程

#### select

1. 线性扫描，不断遍历内容，效率低下
1. 能够监视文件描述符的数量存在最大限制

#### epoll

1. 每当 fd 就绪，采用系统的回调函数直接将 fd 放入，效率高
1. 无最大连接限制

### Crontab

1. 单台服务器 Crontab 脚本太多会导致脚本中断
1. Crontab 任务数量多应分配到负载均衡的服务器上
1. flock 保障脚本不重复进行

### 代码规范

- 规范
  - 函数名明确数据操作（CURD）, 避免产生模糊歧义
  - 变量、返回值 命名规范，体现类型
  - 对废弃函数进行标注
  - 函数单一职责
  - 函数传入参数减少重复参数，函数内部减少参数引用
  - 参数、返回值类型明确，尽量避免 mixed
  - 避免在循环内做运算、判断
  - 相同功能模块的表使用统一的表前缀
- 安全
  - 去重、考虑 null
  - 重视代码对性能的影响
  - 项目上线前检查语法错误
  - 测试覆盖不全，没有运行到所有代码，可能存在隐藏 bug

- 写时复制：多个变量可能指向同一个 value，然后通过 refcount 统计引用数，这时候如果其中一个变量试图更改 value 的内容则会重新拷贝一份 value 修改，同时断开旧的指向，写时复制的机制在计算机系统中有非常广的应用，它只有在必要的时候（写）才会发生硬拷贝，可以很好的提高效率
- 有一个保存配置的 array 数组，这个配置信息很多，数组占用内存也会多一些。当我们向函数传值的时候你是不是考虑过会多占用一个配置数组内存的问题。毕竟函数传值使用的是 copy 的方式。看到写时复制，你就大可放心，完全可以直接把整个数组传进去，只要在函数内部没有对数组进行写入操作，那就是零拷贝，只是使原数组的引用计数值 +1，不会有内存上涨问题
- 垃圾回收：unset 一个变量之后之后由于数组中有子元素指向该变量，所以 refcount > 0，无法通过简单的 gc 机制回收，这种变量就是垃圾，垃圾回收器要处理的就是这种情况，目前垃圾只会出现在 array、object 两种类型中，所以只会针对这两种情况作特殊处理：当销毁一个变量时，如果发现减掉 refcount 后仍然大于 0，且类型是 IS_ARRAY、IS_OBJECT 则将此 value 放入 gc 可能垃圾双向链表中，等这个链表达到一定数量后启动检查程序将所有变量检查一遍，如果确定是垃圾则销毁释放

### Clone

```php
<?php
class B{
 public $val = 10;
}
class A{
 public $val = 20;
 public $b;
 public function __construct(){
  $this->b = new B();
 }
}
$obj_a = new A();
$obj_b = clone $obj_a;
$obj_a->val = 30;
$obj_a->b->val = 40;
var_dump($obj_a);
echo '<br>';
var_dump($obj_b);
/**
object(A)[1]
 public 'val' => int 30
 public 'b' =>
 object(B)[2]
  public 'val' => int 40

object(A)[3]
 public 'val' => int 20
 public 'b' =>
 object(B)[2]
  public 'val' => int 40  b->val受影响
*/
```

```php
<?php
class B{
 public $val = 10;
}
class A{
 public $val = 20;
 public $b;
 public function __construct(){
  $this->b = new B();
 }

 public function __clone(){
  $this->b = clone $this->b; //深度克隆
 }
}
$obj_a = new A();
$obj_b = clone $obj_a;
$obj_a->val = 30;
$obj_a->b->val = 40;
var_dump($obj_a);
echo '<br>';
var_dump($obj_b);
/**
object(A)[1]
 public 'val' => int 30
 public 'b' =>
 object(B)[2]
  public 'val' => int 40

object(A)[3]
 public 'val' => int 20
 public 'b' =>
 object(B)[4]
  public 'val' => int 10  b->val不受影响
*/
```

### 循环优化

```php
    $options = [];
    foreach ($configurationSources as $source) {
        $options = array_merge($options, $source->getOptions());
    }

    //优化，节省内存（-75%）、时间（-99%）
    $options = [];
    foreach ($configurationSources as $source) {
        $options[] = $source->getOptions();
    }

    /* PHP 版本低于 5.6 */
    $options = call_user_func_array('array_merge', $options);
    /* PHP 版本高于 5.6 */
    $options = array_merge([], ...$options); // 空数组覆盖没有循环的情况

    /* PHP 7  */
    $options = array_merge(...$options);
```

### static

静态：从程序运行开始就实例生成内存，可以直接调用，效率高很多，内存常驻

非静态：实例方法开始时生成内存，在调用时申请零散的内存，效率慢，用完就释放

### strtotime

时间加减计算尽量使用`strtotime`函数，直接使用数字进行加减在有东夏令时时区切换的地区会有问题

### 秒杀

```lua
# redis hash 结构版
#key: itemID
#value: {total: N, ordered: M}

#获取商品库存信息
local counts = redis.call("HMGET", KEYS[1], "total", "ordered");
#将总库存转换为数值
local total = tonumber(counts[1])
#将已被秒杀的库存转换为数值
local ordered = tonumber(counts[2])
#如果当前请求的库存量加上已被秒杀的库存量仍然小于总库存量，就可以更新库存
if ordered + k <= total then
    #更新已秒杀的库存量
    redis.call("HINCRBY",KEYS[1],"ordered",k)
    return k                             
end
return 0


# 分布式锁
//使用商品ID作为key
key = itemID
//使用客户端唯一标识作为value
val = clientUniqueID
//申请分布式锁，Timeout是超时时间
lock =acquireLock(key, val, Timeout)
//当拿到锁后，才能进行库存查验和扣减
if(lock == True) {
   //库存查验和扣减
   availStock = DECR(key, k)
   //库存已经扣减完了，释放锁，返回秒杀失败
   if (availStock < 0) {
      releaseLock(key, val)
      return error
   }
   //库存扣减成功，释放锁
   else{
     releaseLock(key, val)
     //订单处理
   }
}
//没有拿到锁，直接返回
else
   return
```
