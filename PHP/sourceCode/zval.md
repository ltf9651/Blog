# ZVAL

* 可以表示 PHP 里任一变量

``` c
// zend_types.h

struct zval_struct {
    zend_value  value;  // 8字节
    union  u1;  // 4字节
    union  u2;  // 4字节
}

// zend_value type多，可表示多类型
// 底层实现区分类型
typedef union _zend_value {
    zend_long  lval;
    double  dval;
    zend_refcounted  *counted;
    zend_string  *string;
    zend_array  *arr;
    zend_object  *obj;
    zend_resource  *res;
    zend_reference *ref;
    zend_function  *func;
    ...
} zend_value;

union {
    struct {
        ZEND_ENDIAN_LOHI_4 (
            zend_uchar  type, // 区分类型
            zend_uchar  type_flags, // 变量类型的特有标记
            zend_uchar  const_flags, // 常量类型标记
            zend_uchar  reserved  // 保留字段
        )
    } v;
    uint32_t  type_info; // -> type
} u1;

/**
zend_uchar type:
    #define IS_UNDEF = 0
    #define IS_NULL = 1
    #define IS_ARRAY/ IS_INT/ IS_LONG ...
*/

union {
    uint32_t  next;  // 解决hash冲突
    uint32_t  cache_slot; // 运行时缓存
    uint32_t  lineno;  // 标记行（ast节点）
    uint32_t  num_args; // 函数参数个数
    uint32_t  fe_pos; // foreach的位置
    uint32_t  fe_iter_idx; // foreach游标的索引位置
    uint32_t  access_flags; // public/protect/private
    uint32_t  property_guard; // 防止类中魔术方法的循环引用
} u2;
```

* 字符串

``` c
// 写时复制
struct _zend_string {
	zend_refcounted_h gc;  // 垃圾回收
	zend_ulong        h;  // 字符串对应hash值，用于数组
	size_t            len;  // 字符串长度
	char              val[1]; // 字符串内容
};

typedef struct _zend_refcounted_h {
	uint32_t         refcount;  // 引用次数   常量字符串：0  /  变量字符串：1
	union {
		struct {
			ZEND_ENDIAN_LOHI_3(
				zend_uchar    type,  // 常量字符串：6
				zend_uchar    flags,  // 常量字符串：2
				uint16_t      gc_info)
		} v;
		uint32_t type_info;
	} u;
} zend_refcounted_h;
```

* 引用

``` php
$a = 'hi';
$b = &$a;  // 此时 $a 和 $b 的type = IS_REFERENCE，refcount = 2，_zend_reference包含的zval type = IS_STRING
$b = 'hello'; //$a 指向的 zval 内容改变
unset($b); // $b type = IS_NULL，$a 指向的地址和内容均不变
echo $a; // hello
```

``` c
struct _zend_reference {
	zend_refcounted_h gc;
	zval              val;
};
```

* 数组

``` c
// hashTable
struct _zend_array {
	zend_refcounted_h gc;
	union {
		struct {
			ZEND_ENDIAN_LOHI_4(
				zend_uchar    flags,
				zend_uchar    nApplyCount,
				zend_uchar    nIteratorsCount,
				zend_uchar    consistency)
		} v;
		uint32_t flags;
	} u;
	uint32_t          nTableMask; // 计算索引值
	Bucket           *arData; // 存 key-value 对
	uint32_t          nNumUsed; // 已使用的空间
	uint32_t          nNumOfElements; // 元素个数
	uint32_t          nTableSize; // 数组大小，默认8，扩容 8 -> 16 -> 32
	uint32_t          nInternalPointer;
	zend_long         nNextFreeElement; // $array[] = 1
	dtor_func_t       pDestructor;
};

/*
 * HashTable Data Layout
 * =====================
 *
 *                 +=============================+
 *                 | HT_HASH(ht, ht->nTableMask) |
 *                 | ...                         |
 *                 | HT_HASH(ht, -1)             |
 *                 +-----------------------------+
 * ht->arData ---> | Bucket[0]                   |
 *                 | ...                         |
 *                 | Bucket[ht->nTableSize-1]    |
 *                 +=============================+

 1、对于key是数字的，就不用涉及到hash运算，此时使用的是packed array；
 如果key的值较大，或者间隔较大，还是会退化成hash array;
 packed array 能够节省索引部分占用的内存，是一个性能上的优化；

2、对于key是非数字的，必须用hash算法进行计算出来它所在bucket的位置，那么索引数组是必不可少的，只能是hash array。
 */
```
