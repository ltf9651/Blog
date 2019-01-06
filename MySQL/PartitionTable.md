## 分区表
#### 分区表在逻辑上为一个表，物理上为多个文件
- table.frm: 元数据
- table.ibd: InnoDB数据文件
  - table#p#p0.ibd
  - table#p#p1.ibd
  - table#p#p2.ibd
  - table#p#pn.ibd

1. **HASH分区**
    > 根据MOD（分区键、分区数）的值把数据行存储到表的不同分区中，数据可平均的分布在各区中，HASH分区键必须为`INT`类型或可通过函数转为`INT`

    > 利用HASH函数打散某列使数据均匀分布，一般用于均衡I/O，缺点是如果hash列上数值有太多的重复值，数据会分布不均匀，不容易管理

    ```sql
    PARTITION BY HASH(id) PARTITION 5;
    ```

1. **RANGE分区**
    > 根据分区键值的范围进行分区，各个分区范围应保持连续且不可出现重叠，默认使用`VALUES LESS THAN`属性，其分区不包括其指定值

    > 适用于分区键为日期的表，查询条件带分区键方便筛选

    ```sql
    PARTITION p0 VALUES LESS THAN (1000);
    PARTITION p1 VALUES LESS THAN (2000);
    PARTITION p2 VALUES LESS THAN MAXVALUE;
    ```

1. **LIST分区**
    > 按分区键取值的列表作为分区，各分区列表不重复，每一行插入数据找到对应的分区列表

    > 一般用于数据枚举

    ```sql
    PARTITION BY LIST(city_name)(
        PARTITION p0 VALUES in ('China', 'Japan'),
        PARTITION p1 VALUES in ('England', 'German')
        PARTITION p3 VALUES in (DEFAULT)
    )
    ```