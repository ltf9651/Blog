### 内存管理

``` C
void *ptr = malloc(size); // 申请的时候传入size，返回指针ptr，指向可用内存首地址

free(ptr); // 释放内存
```

* chunk: 2MB大小的内存
* page: 4kb大小的内存
* chunk可分成512个page
* 分配函数：_emalloc()  根据size大小做不同处理
* 内存与分配：使用mmap分配chunk
* 内存类型
    - Small(30种规格，size <= 3KB) `zend_mm_alloc_small -> zend_mm_alloc_small_slow` 
    - Large(3KB < size <= 2MB - 4KB) `zend_mm_alloc_large -> zend_mm_alloc_pages` 
    - Huge(size> 2MB - 4kB) `zend_mm_alloc_huge -> zend_mm_chunk_alloc` 
* 内存分配流程：emalloc -> zend_mm_alloc_heap(根据size大小选择内存类型) 

### Chunk 内存对齐

* 根据首地址得到内存类型，可通过次地址快速知道内存大小
  + large 内存为 4K 整数倍
  + huge 内存为 2M 整数倍
* 内存申请时会先划定规格，提供能cover的最小规格 ZEND_MM_BINS_INFO
* 如果 free_slot 有空间，直接从 free _slot 取

``` C
static void *zend_mm_chunk_alloc_int(size_t size, size_t alignment)
{
    void *ptr = zend_mm_mmap(size); // 申请地址
    // 如果内存不对齐重新申请
}

typedef struct _zend_alloc_globals {
	zend_mm_heap *mm_heap;
} zend_alloc_globals;

# define AG(v) (alloc_globals.v)
static zend_alloc_globals alloc_globals;

// main_chunk 双向链表
struct _zend_mm_chunk {
	zend_mm_heap      *heap;  // AG zend_mm_heap
	zend_mm_chunk     *next;
	zend_mm_chunk     *prev;
	uint32_t           free_pages;				/* number of free pages */
	uint32_t           free_tail;               /* number of free pages at the end of chunk */
	uint32_t           num;
	char               reserve[64 - (sizeof(void*) * 3 + sizeof(uint32_t) * 3)];
	zend_mm_heap       heap_slot;               /* used only in main chunk */
	zend_mm_page_map   free_map;                /* 512 bits or 64 bytes zend_mm_bitset */
	zend_mm_page_info  map[ZEND_MM_PAGES];      /* 2 KB = 512 * 4 */
};
```

