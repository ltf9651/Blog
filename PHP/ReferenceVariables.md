## 引用变量
定义：用不同的名字访问同一变量内容
 * 以下两段代码$b与$a均指向同一内存空间
 * **Tips**: 使用 unset() 只会取消引用，不会销毁内存空间
```php
$a = 1;
$b = $a;
```
```php
$a = 1;
$b = &$a;
```

* **Example**
```php
$data = ['a', 'b', 'c'];
foreach ($data as $key => $val) {
    $val = &$data[$key];
    print_r($data);
}

/*
loop_1
$key = 0
$val = 'a'
$val = &$data[0] =>'a' (此时$val与$data[0]指向同一地址)
$data = ['a','b','c'];

loop_2
$key = 1
$val = 'b' => $data[0] = 'b' ($data[0]与$val地址相同，$val改变，$data[0]的值也发生改变)
$val = &$data[1] => $val = 'b'（$val地址再次发生改变，与$data[1]相同）
$data = ['b','b','c'];

loop_3
$key = 2
$val = 'c' => $data[1] = 'c'($data[1]与$val地址相同，$val改变，$data[1]的值也发生改变)
$val = &$data[2] => $val =>'c'（$val地址再次发生改变，与$data[2]相同）
$data = ['b','c','c'];
*/
```