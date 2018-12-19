## Tips
* 浮点
```
$a = 0.7;
$b = 0.1;
var_dump(($a + $b) == 0.8); // return false
var_dump(bcadd($a, $b, 2) == 0.8); // return
```

- 标量
  + Bool
  ``` 
  0 == 0.0 == '' == '0' == false == array() == null
  ```
  - 常量定义
    + const：语言结构，可以定义类常量，速度更快
    + define：函数，编译前处理

```
null-- => null
null++ => 1
true(false) ++/-- => true(false)
```

* **Static**：变量值初始化一次（保留处理后的值）

* **include**: 找不到文件时警告，仍继续执行脚本
* **require**：致命错误

```
$array = array('k1' => 'v1', 'k2' => 'v2');
extract($array);
echo $k1 . ' and ' . $k2; //return 'v1 and v2'
```