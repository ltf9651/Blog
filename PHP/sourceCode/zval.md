# ZVAL

* 可以表示PHP里任一变量

``` c
struct zval_struct {
    zend_value  value;  // 8字节
    union  u1;  // 4字节
    union  u2;  // 4字节
}

// zend_value type多，可表示多类型
// 底层实现非弱类型
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
        ZEND_EDIAN_LOHI_4 (
            zend_uchar  type, // 区分类型
            zend_uchar  type_flags,
            zend_uchar  const_flags,
            zend_uchar  reserved
        )
    } v;
    uint32_t  type_info; // -> type
}u1;

/**
zend_uchar type:
    #define IS_UNDEF = 0
    #define IS_NULL = 1
    #define IS_ARRAY/ IS_INT/ IS_LONG ...
*/

union {
    uint32_t  next;
    uint32_t  cache_slot;
    uint32_t  lineno;
    uint32_t  num_args;
    uint32_t  fe_pos;
    uint32_t  fe_iter_idx;
    uint32_t  access_flags;
    uint32_t  property_guard;
} u2;
```

