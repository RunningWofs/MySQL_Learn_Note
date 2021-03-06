# 表

1. InnoDB逻辑存储结构

   InnoDB存储引擎的逻辑结构，数据都被逻辑存放到一个空间中，即**表空间**。表空间由三部分组成，  

   分别是段、区、页（存放数据行）。

   ![avatar](.\pic\InnoDB逻辑存储结构.png)

   1. 表空间

      InnoDB默认有一个共享表空间ibdata1。  
      
      启用innodb_file_per_table后，每张表都有独立的表空间，存放数据、索引、插入缓冲Bitmap页；但是其他数据如回滚（undo）信息，插入缓存索引页、系统事务信息、二次写缓冲等还是存放在共享表中。ps：共享表的大小只会增加，回滚信息哪怕回滚后，共享表占用的存储空间不会减小，只会在内部标记哪些空间可用，下次插入时直接复用可用空间。  
      
   2. 段

      常见的段有<u>数据段、索引段、回滚段</u>等。

      数据段就是B+树的叶子节点，索引段就是B+树的非索引节点。

   3. 区

      区是由连续的页组成的空间，在任何情况下，区的大小都是**1MB**。  

      为了保证区的连续性，InnoDB会一次从磁盘申请4~5个区。默认情况下，一页的大小为16kb，那么，一个区里面就有<u>1MB / 16KB = **64**</u>个页。  

   4. 页

      常见的页类型有：

      * 数据页（B-tree Node）
      * undo页（undo Log Page）
      * 系统页（System Page）
      * 事务数据页
      * 插入缓冲位图页
      * 插入缓冲空闲列表页
      * 未压缩的二进制大对象页
      * 压缩的二进制大对象页

   5. 行

      一页大小默认16kb，InnoDB对行记录有硬性定义，一页最多存储16kb / 2 - 200 = 7992行记录。  

2. InnoDB行记录格式

   1. <u>Compact</u>和<u>Redundant</u>行记录格式

      * Compact

        RowId（如果该表没有主键） |事务Id列 | 回滚指针列 | 变长字段长度列表 | NULL标志位 | 记录头信息 | 列1数据 | 列2数据 | ....... 

        ps：变长字段长度列表按照列的顺序逆序放置    03 02 01 分别代表长度为3、2、1                                                

        ![avatar](.\pic\Compact记录头信息.png)

        NULL标志位记录为空的列，行记录不存储为NULL的列

      * Redundant

        RowId（如果该表没有主键） |事务Id列 | 回滚指针列 | 字段长度偏移列表 | 记录头信息 | 列1数据 | 列2数据 | ......

        ps：字段长度偏移列表逆序存储 

        ![avatar](.\pic\Redundant记录头信息.png)

        由于没有NULL标志位，因此对部分类型的列会实际存储为NULL值，如char定长字段，varchar不存储

   2. Compressed和Dynamic行记录格式

      在Compressed格式里，对于BLOB等数据采取完全行溢出存储，数据页只会存储一个20字节的指针指向Off Page，并且在外部页使用压缩算法大长度类型数据。

3. InnoDB数据页结构

   ![avatar](.\pic\InnoDB数据页结构.png)

   1. File Header

      ![avatar](.\pic\FileHeader 组成部分.png)

   2. Page Header

      ![avatar](./pic/Page Header组成部分.png)
   
   3. Infimun和Supermum Records
   
      是两个特殊的行记录，用来限定记录的边界，Infimun是比该页任何主键值都要小的值，Supermun则是都要大的值。在页内他们就是只有一列的两个特殊的行。在页数据堆里是最顶上的两个记录。
   
   4. User Records和Free Space  
   
      User Records：行记录；
   
      Free Space：空闲空间，链表结构；
   
   5. Page Directory  
   
      将页内的数据划分成多个槽，使用槽进行二分查找可以快速定位行记录所在的槽，然后再顺序遍历找到对应的记录。存储这些槽的地方就是Page Directory。形象点就是一把100cm的尺子，按规则划分为5段，分别为0-20，21-40，41-60，61-80，81-100，那么这些槽（slot）就是0,21,41这种标志性的边界，对这些边界进行二分查找可以快速定位，缩小查找范围。在Page Directory里这些slot就是这些槽的边界行的相对地址。
   
   6. File Trailer
   
      检测页的完整性。  