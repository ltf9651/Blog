## SPL 库

### 双向链表

对于双链表中的每个节点，不仅仅存储自己的信息，还要保存前驱和后继节点的地址

```php
SplDoublyLinkedList implements Iterator , ArrayAccess , Countable {    /* 方法 */
    public __construct ( void )
    public void add ( mixed $index , mixed $newval )
    public mixed bottom ( void )//双链表的尾部节点(最先添加到链表的节点)
    public int count ( void )//双联表元素的个数
    public mixed current ( void )//当前记录
    public int getIteratorMode ( void ) //获取迭代模式
    public bool isEmpty ( void )//检测双链表是否为空
    public mixed key ( void )//当前节点索引
    public void next ( void )//移到下条记录
    public bool offsetExists ( mixed $index )//指定index处节点是否存在
    public mixed offsetGet ( mixed $index )//获取指定index处节点值
    public void offsetSet ( mixed $index , mixed $newval )//设置指定index处值
    public void offsetUnset ( mixed $index )//删除指定index处节点
    public mixed pop ( void )//从双链表的尾部弹出元素
    public void prev ( void )//移到上条记录
    public void push ( mixed $value )//添加元素到双链表的尾部
    public void rewind ( void )//将指针指向迭代开始处
    public string serialize ( void )//序列化存储
    public void setIteratorMode ( int $mode )//设置迭代模式
    public mixed shift ( void )//双链表的头部移除元素
    public mixed top ( void )//双链表的头部节点(最后添加到链表的节点)
    public void unserialize ( string $serialized )//反序列化
    public void unshift ( mixed $value )//双链表的头部添加元素
    public bool valid ( void )//检查双链表是否还有节点
}

$list = new SplDoublyLinkedList();
$list->push('a');
$list->push('b');
$list->push('c');
$list->push('d');

$list->unshift('top');
$list->shift();

$list->rewind();//rewind操作用于把节点指针指向Bottom所在的节点
echo 'curren node:'.$list->current()."<br />";//获取当前节点

$list->next();//指针指向下一个节点
echo 'next node:'.$list->current()."<br />";

$list->next();
$list->next();
$list->prev();//指针指向上一个节点
echo 'next node:'.$list->current()."<br />";

if($list->current())
    echo 'current node is valid<br />';
else
    echo 'current node is invalid<br />';

if($list->valid())//如果当前节点是有效节点，valid返回true
    echo "valid list<br />";
else
  echo "invalid list <br />";

var_dump(array(
    'pop' => $list->pop(),
    'count' => $list->count(),
    'isEmpty' => $list->isEmpty(),
    'bottom' => $list->bottom(),
    'top' => $list->top()
));

$list->setIteratorMode(SplDoublyLinkedList::IT_MODE_FIFO);
var_dump($list->getIteratorMode());

for($list->rewind(); $list->valid(); $list->next()){
    echo $list->current().PHP_EOL;
}

var_dump($a = $list->serialize());
//print_r($list->unserialize($a));

$list->offsetSet(0,'new one');
$list->offsetUnset(0);
var_dump(array(
    'offsetExists' => $list->offsetExists(4),
    'offsetGet' => $list->offsetGet(0),
));
var_dump($list);

//堆栈，先进后出
$stack = new SplStack();//继承自SplDoublyLinkedList类

$stack->push("a<br />");
$stack->push("b<br />");

echo $stack->pop();
echo $stack->pop();
echo $stack->offsetSet(0,'B');//堆栈的offset=0是Top所在的位置，offset=1是Top位置节点靠近bottom位置的相邻节点，以此类推

$stack->rewind();//双向链表的rewind和堆栈的rewind相反，堆栈的rewind使得当前指针指向Top所在的位置，而双向链表调用之后指向bottom所在位置
echo 'current:'.$stack->current().'<br />';
$stack->next();//堆栈的next操作使指针指向靠近bottom位置的下一个节点，而双向链表是靠近top的下一个节点
echo 'current:'.$stack->current().'<br />';
echo '<br /><br />';

//队列，先进先出
$queue = new SplQueue();//继承自SplDoublyLinkedList类
$queue->enqueue("a<br />");//插入一个节点到队列里面的Top位置
$queue->enqueue("b<br />");
$queue->offsetSet(0,'A');//队列的offset=0是Bottom所在的位置，offset=1是Bottom位置节点靠近top位置的相邻节点，以此类推
echo $queue->dequeue();
echo $queue->dequeue();
echo "<br /><br />";
```

### 堆

堆 (Heap) 就是为了实现优先队列而设计的一种数据结构，它是通过构造二叉堆（二叉树的一种）实现。根节点最大的堆叫做最大堆或大根堆（SplMaxHeap），根节点最小的堆叫做最小堆或小根堆（SplMinHeap）。二叉堆还常用于排序（堆排序）

```php
abstract SplHeap implements Iterator , Countable {
    /* 方法 用法同双向链表一致 */
    public __construct ( void )
    abstract protected int compare ( mixed $value1 , mixed $value2 )
    public int count ( void )
    public mixed current ( void )
    public mixed extract ( void )
    public void insert ( mixed $value )
    public bool isEmpty ( void )
    public mixed key ( void )
    public void next ( void )
    public void recoverFromCorruption ( void )
    public void rewind ( void )
    public mixed top ( void )
    public bool valid ( void )
}

class MySplHeap extends SplHeap{
    //compare()方法用来比较两个元素的大小，绝对他们在堆中的位置
    public function compare( $value1, $value2 ) {
        return ( $value1 - $value2 );
        }
}

$obj = new MySplHeap();
$obj->insert(0);
$obj->insert(1);
$obj->insert(2);
$obj->insert(3);
$obj->insert(4);
echo $obj->top();//4
echo $obj->count();//5

foreach ($obj as $item) {
    echo $item."<br />";
}
```

### 阵列

优先队列也是非常实用的一种数据结构，可以通过加权对值进行排序，由于排序在 php 内部实现，业务代码中将精简不少而且更高效。通过 `SplPriorityQueue::setExtractFlags(int  $flag)` 设置提取方式可以提取数据（等同最大堆）、优先级、和两者都提取的方式

```php
SplFixedArray implements Iterator , ArrayAccess , Countable {
　　/* 方法 */
　　public __construct ([ int $size = 0 ] )
　　public int count ( void )
　　public mixed current ( void )
　　public static SplFixedArray fromArray ( array $array [, bool $save_indexes = true ] )
　　public int getSize ( void )
　　public int key ( void )
　　public void next ( void )
　　public bool offsetExists ( int $index )
　　public mixed offsetGet ( int $index )
　　public void offsetSet ( int $index , mixed $newval )
　　public void offsetUnset ( int $index )
　　public void rewind ( void )
　　public int setSize ( int $size )
　　public array toArray ( void )
　　public bool valid ( void )
　　public void __wakeup ( void )
}

$arr = new SplFixedArray(4);
$arr[0] = 'php';
$arr[1] = 1;
$arr[3] = 'python';//遍历， $arr[2] 为null
foreach($arr as $v) {
    echo $v . PHP_EOL;
}

//获取数组长度
echo $arr->getSize(); //4

//增加数组长度
$arr->setSize(5);
$arr[4] = 'new one';

//捕获异常
try{
    echo $arr[10];
} catch (RuntimeException $e) {
    echo $e->getMessage();
}
```

### 映射

存储对象

```php
SplObjectStorage implements Countable , Iterator , Serializable , ArrayAccess {
　　/* 方法 */
　　public void addAll ( SplObjectStorage $storage )
　　public void attach ( object $object [, mixed $data = NULL ] )
　　public bool contains ( object $object )
　　public int count ( void )
　　public object current ( void )
　　public void detach ( object $object )
　　public string getHash ( object $object )
　　public mixed getInfo ( void )
　　public int key ( void )
　　public void next ( void )
　　public bool offsetExists ( object $object )
　　public mixed offsetGet ( object $object )
　　public void offsetSet ( object $object [, mixed $data = NULL ] )
　　public void offsetUnset ( object $object )
　　public void removeAll ( SplObjectStorage $storage )
　　public void removeAllExcept ( SplObjectStorage $storage )
　　public void rewind ( void )
　　public string serialize ( void )
　　public void setInfo ( mixed $data )
　　public void unserialize ( string $serialized )
　　public bool valid ( void )
}

class A {
    public $i;
    public function __construct($i) {
        $this->i = $i;
    }
}

$a1 = new A(1);
$a2 = new A(2);
$a3 = new A(3);
$a4 = new A(4);

$container = new SplObjectStorage();

//SplObjectStorage::attach 添加对象到Storage中
$container->attach($a1);
$container->attach($a2);
$container->attach($a3);

//SplObjectStorage::detach 将对象从Storage中移除
$container->detach($a2);

//SplObjectStorage::contains用于检查对象是否存在Storage中
var_dump($container->contains($a1)); //true
var_dump($container->contains($a4)); //false

//遍历
$container->rewind();
while($container->valid()) {
    var_dump($container->current());
    $container->next();
}
```

### MultipleIterator

```php
<?php
$idIter = new ArrayIterator(array('1', '2'));
$nameIter = new ArrayIterator(array('张三', '李四'));
$ageIter = new ArrayIterator(array(12, 56));
$mit = new MultipleIterator(MultipleIterator::MIT_KEYS_ASSOC);
$mit->attachIterator($idIter, 'id');
$mit->attachIterator($nameIter, 'name');
$mit->attachIterator($ageIter, 'age');

foreach ($mit as $k => $value) {
    print_r($value);
}
```

### FileSystemIterator

```php
<?php
date_default_timezone_set('PRC');
$it = new FilesystemIterator('.');
foreach ($it as $finfo) {
    echo $finfo->getFilename();
    echo $finfo->getSize();
    echo date('Y-m-d H:i:s', $finfo->getMtime()) . PHP_EOL;
}
```

### SplFileInfo

```php
date_default_timezone_set('PRC');
$file = new SplFileInfo('tmp.txt');
$file->getCTime();
$file->getMTime();
$fileObj = $file->openFile('r');
while ($fileObj->valid()) {
    echo $fileObj->fgets();
}
$fileObj = null; // fclose
$file = null;
```
