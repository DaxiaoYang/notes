# Database System

[TOC]

课程概要：

- 关系型数据库

- 存储

- 执行

- 并发控制

- 数据恢复

- 分布式数据库

  



## Relational model



数据库：

Organized collection of inter-related data that models some aspects of out real world





数据模型

- **关系型**：绝大部分的DBMS都是

- key-value

- 图

- 文档

- 列式

- 数组/矩阵

  



关系模型是抽象 数据库实现是具体 两者分离开来了



关系与表对应

元组与一个行数据对应



关系代数：是基于集合运算

- 过滤：对行
- 投影：对列

- 交
- 并
- 差
- 乘积
- natural join





## Storage1

storage1和storage2回答的问题是：

> DBMS怎么用磁盘上的文件来表示数据库



存储主要关注的是可持久化设备-磁盘上的数据的存储



数据库管理系统需要进行的操作：磁盘与内存之间的IO



DBMS的存储管理器需要记录页中的数据以及空闲位置



页：大小固定的块，通常一个块只存储一种类型的数据：元组、日志、索引



堆文件：页的无序组合，用于定位一个页

实现：

- 链表：两个表头，分别指向空闲页与数据页
- 页目录：是一个特殊的页，存储了页的位置和页中空闲空间的大小



页的组成：

- 页头：记录元数据

  - 页大小
  - 校验和
  - 软件版本
  - 事务可见性

- 数据：两种方式

  - slotted pages：

    由槽数组（->增长） 元组（<-增长） 和空闲空间组成

    ![image-20210815072811667](/Users/yangsiping/Library/Application Support/typora-user-images/image-20210815072811667.png)

  - log-structured：只记录修改数据的操作

    ![image-20210815072839433](/Users/yangsiping/Library/Application Support/typora-user-images/image-20210815072839433.png)



元组的组成：头 + 数据

每个元组都有一个ID=page_id + slot



## Storage2

**数据表示**

元组的数据在存储文件中只是字节信息，需要另外的元信息才能将其解析出来



**两种工作场景**

OLTP：

- 大部分查询只影响某一个元组 对某个元组的增删查改
- 写比读多
- 重复的操作多
- 大部分业务系统刚开始都是这样的



OLAP：

- 读取的数据通常是一张大表的一部分属性
- 对数据进行聚合分析



**存储模型**

行式存储：

元组的所有属性都连续存储在一个页中

- 优点：插入 修改 删除快
- 缺点：当需要对表的一部分数据进行查询时 会将很多不需要的数据也扫描到 并加载到内存中



列式存储：

元组的单一个的属性连续存储在一个页中

- 优点：对只查询表的一部分数据很快 相同属性的数据放在一起方便压缩
- 缺点：查询某个元组的数据不快 修改 删除也不快 

实现：一个页中给当前属性分配的大小存储空间是等长的





## Buffer Pool

**Lock和Latch**

共同点：都是锁的概念

不同点：

- Lock针对的对象是数据库 表 元组这种逻辑上的概念 保证不受其他事务干扰

  需要支持回退

- Latch针对的对象是Buffer Pool中的内部结构 保证不受其他线程干扰

  不需要支持回退



**Buffer Pool**

相当于一个对磁盘中的页的数据的内存缓存

当查找某个页的时候会先查找buffer pool，未命中再去查盘

数据结构就是一个页大小内存块的数组



元数据：

- page table: page_id -> frame
- dirty-flag：当某个页被修改了 必须将修改写回磁盘
- pin counter：访问该frame的线程数 当counter>0 此页不能被替换出去



优化：

- 多个buffer pool 减少锁竞争 指定per transaction的替换策略 提高局部性
- 提前读 常用于顺序读页时
- 多个查询共享游标



分配策略

- 全局: 考虑所有的事务
- 局部：per transaction



**Replacement policy**

替换策略 当需要buffer pool中的frame不够用的时候

需要把页从内存替换出去 此时需要选择替换哪个页



需要达到的目标：

- 正确性：正在访问的页不能替换出去
- 精准性：替换出去的页不会很快地又被读回来
- 用于实现策略的元数据不能太多



LRU

- 为每个页都维持了一个时间戳
- 选择移除时间戳最老的页 



CLOCK：LRU的近似

设置一个访问位有线程访问时置为1 meta-overhead少了

在一个循环队列（环）中扫描 遇1则置为0 遇0则移除



LRU和CLOCK缺点：

- 当有顺序读的查询时 缓存会被污染
- 没有利用DBMS掌握的其他信息来提示哪些页更重要



其他策略：

- LRU-K：利用最后K次引用的信息
- priority-hints: 利用事务告知的信息
- localization: 以单个transaction为单位作查询





