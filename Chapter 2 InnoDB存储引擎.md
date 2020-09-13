# Chapter 2 InnoDB存储引擎  

### 2.3 InnoDB体系架构

	#### 	2.3.1 后台线程

- **Master Thread**

  负责将缓冲池中的数据异步刷新到磁盘，保持数据一致性，包括脏页的刷新、合并插入缓冲、UNDO页回收等；

- **IO Thread**  

  总共有四种类型的IO，read IO, write IO, insert IO, log IO；

- **Purge Thread**

  事务被提交后，其所使用的undolog可能不再需要，因此需要PurgeThread(清理线程)来回收已经使用并分配的undo页；

- **Page Cleaner Thread**

  InnoDB 1.2.X引入，作用是将之前版本的脏页刷新操作放到单独的线程中来完成，目的是减轻原Master Thread的工作及用户查询线程的阻塞，进一步提高InnoDB存储引擎的性能；

  

  *********

  #### 2.3.2 内存

  1. **LRU**

     InnoDB一般通过LRU算法来管理缓冲池。但是与传统的LRU算法有区别。InnoDB的LRU加入了midpoint的概念，在midpoint之前的列表称为new列表，之后为old列表，这样做的原因是：

