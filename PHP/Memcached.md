## Memcached

### 内存管理机制

Slab Allocation ：首先从操作系统申请一大块内存，并将其分割成各种尺寸的块 Chunk，并把尺寸相同的块分成组 Slab Class。其中，Chunk 就是用来存储 key-value 数据的最小单位。每个 Slab Class 的大小，可以在 Memcached 启动的时候通过制定 Growth Factor 来控制。假定 Figure 1 中 Growth Factor 的取值为 1.25，所以如果第一组 Chunk 的大小为 88 个字节，第二组 Chunk 的大小就为 112 个字节，依此类推。

当 Memcached 接收到客户端发送过来的数据时首先会根据收到数据的大小选择一个最合适的 Slab Class，然后通过查询 Memcached 保存着的该 Slab Class 内空闲 Chunk 的列表就可以找到一个可用于存储数据的 Chunk。当一条数据库过期或者丢弃时，该记录所占用的 Chunk 就可以回收，重新添加到空闲列表中。从以上过程我们可以看出 Memcached 的内存管理制效率高，而且不会造成内存碎片，但是它最大的缺点就是会导致空间浪费。因为每个 Chunk 都分配了特定长度的内存空间，所以变长数据无法充分利用这些空间。如图所示，将 100 个字节的数据缓存到 128 个字节的 Chunk 中，剩余的 28 个字节就浪费掉了
