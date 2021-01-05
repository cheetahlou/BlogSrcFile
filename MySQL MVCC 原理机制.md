---
title: MySQL MVCC 原理机制
date: 2019-04-17 23:37:00
tags: 
- MySQL 
- MVCC
- undo log

---



## MySQL MVCC 原理机制

## **什么是 MVCC**

`MVCC (Multiversion Concurrency Control)` 中文全程叫**多版本并发控制**，是现代数据库（包括 `MySQL`、`Oracle`、`PostgreSQL` 等）引擎实现中常用的处理读写冲突的手段，**目的在于提高数据库高并发场景下的吞吐性能**。

如此一来不同的事务在并发过程中，`SELECT` 操作可以不加锁而是通过 `MVCC` 机制读取指定的版本历史记录，并通过一些手段保证保证读取的记录值符合事务所处的隔离级别，从而解决并发场景下的读写冲突。



### InnoDB的MVCC实现

在InnoDB中，主要是通过使用readview的技术来实现判断。查询出来的每一行记录，都会用readview来判断一下当前这行是否可以被当前事务看到，如果可以，则输出，否则就利用undolog来构建历史版本，再进行判断，知道记录构建到最老的版本或者可见性条件满足。

InnoDB的多版本并不是直接存储多个版本的数据，而是所有更改操作利用行锁做并发控制，这样对某一行的更新操作是串行化的，然后用Undo log记录串行化的结果。当快照读的时候，利用Undo log重建需要读取版本的数据，从而实现读写并发。

### 相关概念：

#### InnoDB行数据三个隐藏字段

**`事务ID`(`DB_TRX_ID`)**：

表示最近一次插入或者更新该记录的事务ID

**`回滚指针`(`DB_ROLL_PTR`)**：

指向该记录的回滚段rollback segment的undo log记录，`InnoDB` 便是通过这个指针找到之前版本的数据。该行记录上所有旧版本，在 `undo` 中都通过链表的形式组织

**`DB_ROW_ID`**：

自动递增，当表上没有用户主键的时候，InnoDB会自动产生聚集索引

另外，每条记录的头信息（`record header`）里都有一个专门的 `bit`（**`deleted_flag`**）来表示当前记录是否已经被删除



![img](https://i.imgur.com/GOStr4h.png)

### Undo log

如图所示,undo log中记录之前修改该行数据的事务ID以及被修改的历史数据

整个MVCC的机制都是通过`DB_TRX_ID`,`DB_ROLL_PTR`这2个隐藏字段来实现的

##### 执行过程

- 用排他锁锁定该行
- 记录redo log
- 记录undo log
- 修改当前行的值,修改行的事务ID为当前事务ID
- 回滚指针指向undo log刚刚记录的位置

#### ReadView   （MySQL中等同 快照、snapshot）

在事务开始的时候会根据上面的事务链表构造一个ReadView,初始化方法如下

#### Read View

```C
// readview 初始化
// m_low_limit_id = trx_sys->max_trx_id; 
// m_up_limit_id = !m_ids.empty() ? m_ids.front() : m_low_limit_id;
ReadView::ReadView()
    :
    m_low_limit_id(),
    m_up_limit_id(),
    m_creator_trx_id(),
    m_ids(),
    m_low_limit_no()
{
    ut_d(::memset(&m_view_list, 0x0, sizeof(m_view_list)));
}
```

- ReadView主要结构
  - m_low_limit_id。 事务ID大于等于该值的数据修改不可见
  - m_up_limit_id. 事务ID小于该值的数据修改可见。
  - m_creator_trx_id。创建该ReadView的事务，该事务ID的数据修改可见。
  - m_ids。当快照创建时的活跃读写事务列表。
  - m_low_limit_no。事务number，上一节介绍Undo log时候，事务提交时候获取同时写入Undo log中的值。事务number小于该值的对该ReadView不可见。利用该信息可以Purge不需要的Undo。
  - m_closed。 标记该ReadView closed，用于优化减少trx_sys->mutex这把大锁的使用。

#### 可见性判断

设要读取的行的最后提交事务id(即当前数据行的稳定事务id)为 `trx_id_current`
当前新开事务id为 `new_id`
当前新开事务创建的快照`read view` 中最早的事务id为`up_limit_id`, 最迟的事务id为`low_limit_id`(注意这个low_limit_id=未开启的事务id=当前最大事务id+1)
比较:

- 1.`trx_id_current < up_limit_id`, 这种情况比较好理解, 表示, 新事务在读取该行记录时, 该行记录的稳定事务ID是小于, 系统当前所有活跃的事务, 所以当前行稳定数据对新事务可见, 跳到步骤5.
- 2.`trx_id_current >= trx_id_last`, 这种情况也比较好理解, 表示, 该行记录的稳定事务id是在本次新事务创建之后才开启的, 但是却在本次新事务执行第二个select前就commit了，所以该行记录的当前值不可见, 跳到步骤4。
- 3.`trx_id_current <= trx_id_current <= trx_id_last`, 表示: 该行记录所在事务在本次新事务创建的时候处于**活动**状态，从up_limit_id到low_limit_id进行遍历，如果trx_id_current等于他们之中的某个事务id的话，那么不可见, 调到步骤4,否则表示可见。
- 4.从该行记录的 DB_ROLL_PTR 指针所指向的回滚段中取出最新的undo-log的版本号, 将它赋值该 `trx_id_current`，然后跳到步骤1重新开始判断。
- 5.将该可见行的值返回。

```C
// 判断数据对应的聚簇索引中的事务id在这个readview中是否可见
bool changes_visible(
        trx_id_t        id, // 当前记录行的稳定事务id
        const table_name_t& name) const
        MY_ATTRIBUTE((warn_unused_result))
    {
        ut_ad(id > 0);
    	//如果记录上的trx_id小于read_view_t->up_limit_id，则说明这条记录的最后修改在readview创建之前，因此这条记录可以被看见。
        // 或者当前记录trx_id等于创建该readview的id就是它自己,那么也是可见的
        if (id < m_up_limit_id || id == m_creator_trx_id) {
            return(true);
        }

        check_trx_id_sanity(id, name);
        // 如果记录上的trx_id大于等于read_view_t->low_limit_id，即大于事务链表中的最大值，则说明这条记录的最后修改在readview创建之后，那么不可见
        if (id >= m_low_limit_id) {

            return(false);
        // 如果事务链表是空的,那也是可见的
        } else if (m_ids.empty()) {

            return(true);
        }

        const ids_t::value_type*    p = m_ids.data();
    	//如果记录上的trx_id在up_limit_id和low_limit_id之间，且trx_id在read_view_t->descriptors之中，则表示这条记录的最后修改是在readview创建之时，被另外一个活跃事务所修改，所以这条记录也不可以被看见，需要查找 Undo Log 链得到上一个版本，然后根据该版本的 DB_TRX_ID 再从头计算一次可见性；如果不在，说明创建 ReadView 时生成该版本的事务已经被提交，该版本可以被访问。如果trx_id不在read_view_t->descriptors之中，则表示这条记录的最后修改在readview创建之前，所以可以看到。
        return(!std::binary_search(p, p + m_ids.size(), id));
    }
```

![img](https://pic4.zhimg.com/80/v2-de7ecca682e8134a16c2006e836c20bf_720w.jpg)

#### read view快照的生成时机

 **正是因为生成时机的不同, 造成了RC,RR两种隔离级别的不同可见性**;

- 在innodb中(默认repeatable read级别), 事务在begin/start transaction之后的**第一条select读**操作后, 会创建一个快照(read view), 将当前系统中活跃的其他事务记录记录起来;事务执行过程中，数据的可见性不会变，所以在事务内部不会出现不一致的情况。
- 在innodb中(默认repeatable committed级别), 事务中**每条select**语句都会创建一个快照(read view);



## 多版本控制MVCC

数据库需要做好版本控制，防止不该被事务看到的数据(例如还没提交的事务修改的数据)被看到。在InnoDB中，主要是通过使用readview的技术来实现判断。查询出来的每一行记录，都会用readview来判断一下当前这行是否可以被当前事务看到，如果可以，则输出，否则就利用undolog来构建历史版本，再进行判断，知道记录构建到最老的版本或者可见性条件满足。

在trx_sys中，一直维护这一个全局的活跃的读写事务id(`trx_sys->descriptors`)，id按照从小到大排序，表示在某个时间点，数据库中所有的活跃(已经开始但还没提交)的读写(必须是读写事务，只读事务不包含在内)事务。当需要一个一致性读的时候(即创建新的readview时)，会把全局读写事务id拷贝一份到readview本地(read_view_t->descriptors)，当做当前事务的快照。read_view_t->up_limit_id是read_view_t->descriptors这数组中最小的值，read_view_t->low_limit_id是创建readview时的max_trx_id，即一定大于read_view_t->descriptors中的最大值。当查询出一条记录后(记录上有一个trx_id，表示这条记录最后被修改时的事务id)，可见性判断的逻辑如下(`lock_clust_rec_cons_read_sees`)：

如果记录上的trx_id小于read_view_t->up_limit_id，则说明这条记录的最后修改在readview创建之前，因此这条记录可以被看见。

如果记录上的trx_id大于等于read_view_t->low_limit_id，则说明这条记录的最后修改在readview创建之后，因此这条记录肯定不可以被看家。

如果记录上的trx_id在up_limit_id和low_limit_id之间，且trx_id在read_view_t->descriptors之中，则表示这条记录的最后修改是在readview创建之时，被另外一个活跃事务所修改，所以这条记录也不可以被看见。如果trx_id不在read_view_t->descriptors之中，则表示这条记录的最后修改在readview创建之前，所以可以看到。

基于上述判断，如果记录不可见，则尝试使用undo去构建老的版本(`row_vers_build_for_consistent_read`)，直到找到可以被看见的记录或者解析完所有的undo。

针对RR隔离级别，在第一次创建readview后，这个readview就会一直持续到事务结束，也就是说在事务执行过程中，数据的可见性不会变，所以在事务内部不会出现不一致的情况。针对RC隔离级别，事务中的每个查询语句都单独构建一个readview，所以如果两个查询之间有事务提交了，两个查询读出来的结果就不一样。从这里可以看出，在InnoDB中，RR隔离级别的效率是比RC隔离级别的高。此外，针对RU隔离级别，由于不会去检查可见性，所以在一条SQL中也会读到不一致的数据。针对串行化隔离级别，InnoDB是通过锁机制来实现的，而不是通过多版本控制的机制，所以性能很差。

由于readview的创建涉及到拷贝全局活跃读写事务id，所以需要加上trx_sys->mutex这把大锁，为了减少其对性能的影响，关于readview有很多优化。例如，如果前后两个查询之间，没有产生新的读写事务，那么前一个查询创建的readview是可以被后一个查询复用的。

### 案例

事务对某行记录的简化更新过程：

![img](https://i.imgur.com/MLFnrkq.png)

![img](https://i.imgur.com/vtr5itr.png)

## Purge清理操作

旧版本数据不再被任何view访问就可以被删除了。5.6以上版本支持独立purge线程，用户可以通过参数Innodb_purge_threads设置purge线程个数。

有两类purge线程：

- coordinator thread：srv_purge_coordinator_thread，全局只有1个
- worker thread：srv_worker_thread，系统有innodb_purge_threads - 1个

coordinator thread负责启动worker thread参与到purge工作中。

增加purge线程的策略是：trx_sys->rseg_history_len比上次循环变大了或者rseg_history_len超过某一阈值，需要引进更多的worker thread。

减少purge线程的策略是：如果之前使用多个purge 线程，trx_sys->rseg_history_len并没有变大，可能需要减少worker thread。

在进行purge之前，首先要确定purge线程要做哪些工作，也就是说哪些undo log可以被purged。

purge也是通过read view来确定工作范围，被称为purge view。如果系统有活跃read view，就选取最老的read view作为purge view。

如果不存在就给trx_sys的状态打个snapshot，作为purge view，可以被purge的undo log其trx_no一定是小于系统中所有已提交事务的trx->no。

这里插一句，在事务commit时，会把产生的trx->no加入到trx_sys->serialisation_list链表，这个链表是按照trx->no升序次序排列，也就是维护了trx commit顺序。

InnoDB初始化的时候会初始化purge_sys数据结构，其中一个工作就是创建purge graph。

这是总共3层结构的图：

- 第1层是fork节点
- 第2次是thrd节点（表示purge thread）
- 第3层是node节点（表示purge task）

所有的thrd节点被链入到fork->thrs链表中；fork地址存储在purge_sys->query，可以通过purge_sys直接访问。

执行purge的时候总是遍历purge_sys->query->thrs链表，给每个purge线程分配purge任务（trx_purge_attach_undo_recs）。

解析undo log的调用路径如下：

```
srv_purge_coordinator_thread -> srv_do_purge -> trx_purge ->
        trx_purge_attach_undo_recs -> trx_purge_fetch_next_rec -> 
               trx_purge_get_next_rec
```

purge_sys->next_stored为FALSE时，表示rseg_iter当前指向的rseg无效，需要把rseg_iter移到下一个有效的rseg（TrxUndoRsegsIterator::set_next）。

purge_sys->purge_queue维护了一个最小堆，每次pop最顶元素，可以得到trx_no最小的rollback segment（TrxUndoRsegsIterator::set_next）。

5.7支持临时表的noredo的rollback segment，set_next遇到redo rollback segment和noredo rollback segment同时存在的情况会一股脑把这两个rollback segment都pop出来加入到 purge_sys->rseg_iter->m_trx_undo_rsegs数组中，也在TrxUndoRsegsIterator::set_next实现。

如果没有rollback segment需要purge话，purge_sys->rseg设置为NULL，purge线程会去睡眠（trx_purge_choose_next_log）。

一般情况下都是有rollback segment需要处理的，purge_sys->rseg更新成purge_sys->rseg_iter->m_trx_undo_rsegs的第1项（至多2项）。

purge_sys中的相应成员也要更新，指向当前rseg上次purge到的位置（TrxUndoRsegsIterator::set_next）。

update undo的del_marks域正常情况下都是TRUE，因为update/delete操作都需要对old value进行标记删除。

如果purge_sys->rseg->last_del_marks是FALSE的话，表示这是一个dummy的undo log，不需要做物理删除。这种情况下，把purge_sys->offset设置成0，做个标记表示这个undo log不需要被purged（trx_purge_read_undo_rec）。

正常情况下purge_sys->rseg->last_del_marks是TRUE，可以通过<purge_sys->rseg->space, purge_sys->hdr_page_no, purge_sys->hdr_offset>读取undo log记录（trx_purge_read_undo_rec）。

并把purge_sys以下四个域设置成undo log记录相应的信息（trx_purge_read_undo_rec）。

```
    purge_sys->offset = offset; /* undo log记录的offset */
    purge_sys->page_no = page_no; /* undo log记录的pageno */
    purge_sys->iter.undo_no = undo_no; /* undo log记录的undo_no，trx内部undo的序列号 */
    purge_sys->iter.undo_rseg_space = undo_rseg_space; /* undo log的tablespace */
```

为了保证purge_sys以上4个域一定是指向下一个有效undo log，每次读取undo log时都会捎带着读取下一个undo log，并把上面这四个域更新为下一个undo log的信息，方面后续访问（trx_purge_get_next_rec）。

如果是dummy undo，trx_purge_get_next_rec会去读prev_undo（trx_purge_rseg_get_next_history_log），用prev_log信息更新rseg中下一个purge信息。

在此之后，还会把rseg->last_trx_no压入最小堆，待后面继续处理这个rseg。 然后调用trx_purge_choose_next_log选择下一个处理的rseg，并读取第一个undo log（trx_purge_get_next_rec）。

就这样挨个读取undo log，trx_purge_attach_undo_recs中有一个大循环，每次调用trx_purge_fetch_next_rec读到一个undo log后，把它存放到purge节点（purge graph的第三级节点） node->undo_recs数组里面，循环下一次执行切换到下一个thr（purge 线程）。

循环的结束条件是：

- 没有新的undo log
- 处理过的undo log达到batch size（一般是300）

达到循环结束条件后，trx_purge_attach_undo_recs返回。如果n_purge_threads > 1 (需要worker线程参与purge），coordinator线程会以round-robin方式启动n_purge_threads - 1个worker线程。

不管有没有worker线程参与purge，coordinator线程都会调用que_run_threads（在trx_purge上下文）去处理purge任务。

purge任务如何处理呢？通俗的说purge就是删除被标记delete marker的记录项。

大致过程如下：

```
srv_purge_coordinator_thread -> srv_do_purge -> trx_purge ->
        que_run_threads -> que_run_threads_low -> que_thr_step
               row_purge_step -> row_purge -> row_purge_record ->
                       row_purge_del_mark -> row_purge_remove_sec_if_poss
```

一般删除的原则是先删除二级索引再删除clustered索引（row_purge_del_mark）。

另一种情况是聚集索引in-place更新了，但二级索引上的记录顺序可能发生变化，而二级索引的更新总是标记删除 + 插入，因此需要根据回滚段记录去检查二级索引记录序是否发生变化，并执行清理操作（row_purge_upd_exist_or_extern）。

前面提到过在parse undo log时，可能遇到dummy undo log。返回到row_purge执行时需要判读是否是dummy undo，如果是就什么也不做。

#### truncate undo space

trx_purge在处理完一个batch（通常是300）之后，调用trx_purge_truncate_historypurge_sys对每一个rseg尝试释放undo log（trx_purge_truncate_rseg_history）。

大致过程是：把每个purge过的undo log从history list移除，如果undo segment中所有的undo log都被释放，可以尝试释放undo segment，这里隐式释放file segment到达释放存储空间的目的。



### 思考

1. 一般我们认为MVCC有下面几个特点：
   - 每行数据都存在一个版本，每次数据更新时都更新该版本
   - 修改时Copy出当前版本, 然后随意修改，各个事务之间无干扰
   - 保存时比较版本号，如果成功(commit)，则覆盖原记录, 失败则放弃copy(rollback)
   - 就是每行都有版本号，保存时根据版本号决定是否成功，**听起来含有乐观锁的味道, 因为这看起来正是，在提交的时候才能知道到底能否提交成功**
2. 而InnoDB实现MVCC的方式是:
   - 事务以排他锁的形式修改原始数据
   - 把修改前的数据存放于undo log，通过回滚指针与主数据关联
   - 修改成功（commit）啥都不做，失败则恢复undo log中的数据（rollback）
3. **二者最本质的区别是**: 当修改数据时是否要`排他锁定`，如果锁定了还算不算是MVCC？

- Innodb的实现真算不上MVCC, 因为并没有实现核心的多版本共存, `undo log` 中的内容只是串行化的结果, 记录了多个事务的过程, 不属于多版本共存。但理想的MVCC是难以实现的, 当事务仅修改一行记录使用理想的MVCC模式是没有问题的, 可以通过比较版本号进行回滚, 但当事务影响到多行数据时, 理想的MVCC就无能为力了。
- 比如, 如果事务A执行理想的MVCC, 修改Row1成功, 而修改Row2失败, 此时需要回滚Row1, 但因为Row1没有被锁定, 其数据可能又被事务B所修改, 如果此时回滚Row1的内容，则会破坏事务B的修改结果，导致事务B违反ACID。 这也正是所谓的 `第一类更新丢失` 的情况。
- 也正是因为InnoDB使用的MVCC中结合了排他锁, 不是纯的MVCC, 所以第一类更新丢失是不会出现了, 一般说更新丢失都是指第二类丢失更新。





**参考资料：** 

MySQL · 源码分析 · InnoDB的read view，回滚段和purge过程简介：http://mysql.taobao.org/monthly/2018/03/01/

MySQL · 引擎特性 · InnoDB MVCC 相关实现：http://mysql.taobao.org/monthly/2018/11/04/

MySQL · 引擎特性 · InnoDB 事务系统：http://mysql.taobao.org/monthly/2017/12/01/

MVCC原理探究及MySQL源码实现分析：https://blog.csdn.net/woqutechteam/article/details/68486652

MySQL-InnoDB-MVCC多版本并发控制：https://segmentfault.com/a/1190000012650596