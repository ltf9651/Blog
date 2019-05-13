## InnoDB

InnoDB 将表中的数据存储到磁盘上，而真正处理数据的过程是发生在内存中，在处理写请求的时候还需将内存中的数据刷入磁盘。

读取数据时，InnoDB 将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，InnoDB中页的大小一般为 16 KB。一般情况下，一次最少从磁盘中读取16KB的内容到内存中，一次最少把内存中的16KB内容刷新到磁盘中

一个页一般是16KB，当记录中的数据太多，当前页放不下的时候，会把多余的数据存储到其他页中，这种现象称为行溢出

### InnoDB 行格式

#### COMPACT
![](https://user-gold-cdn.xitu.io/2019/3/12/169710e8fafc21aa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### Redundant
![](https://user-gold-cdn.xitu.io/2019/3/12/169710e99a69ba3d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### Dynamic和Compressed
![](https://user-gold-cdn.xitu.io/2019/3/12/169710e9b2c2b71e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
