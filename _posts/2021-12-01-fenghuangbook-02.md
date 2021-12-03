---
layout:     post
title:      读薄凤凰架构系列 - 事务2
subtitle:    "\"Transaction\""
date:       2021-11-05
author:     Lin
header-img: img/post-bg-black-engineer0524.jpg
catalog: true
tags:
    - architecture
---

## **全局事务**

单个服务使用多个数据源场景下，本地事务(数据库事务)是无法解决事务强一致性问题。

还是前面的场景，账单、库存和资金 分别是不同的数据库，事务还是使用Spring的声明式 -- `@Transactional`注解，那么，实际上是使用了多个本地事务 -- 分别为 账单数据库的事务、库存数据库的事务、资金数据库的事务。假设 账单事务和库存事务都commit()后，操作库存时出现错误，是无法rollback前两个事务。

伪代码:

```java
public void executeBySettlement(Settlement bill){
    billTransaction.begin();
    warehouseTransaction.begin();
    financeTransaction.begin();
    try {
        billService.pay(bill.getMoney());
        warehouseService.deliver(bill.getItems());
        financeAccountService.receipt(bill.getMoney());
        billTransaction.commit();
        warehouseTransaction.commit();
        financeTransaction.commit();
    } catch(Exception e) {
        billTransaction.rollback();
        warehouseTransaction.rollback();
        financeTransaction.rollback();
    }
}
```

全局事务是一种解决方案。

> 依据《凤凰架构》的定义，全局事务仅应用于单服务多数据源的场合中，对于多节点而且互相调用彼此服务的场合（典型的就是现在的微服务系统）是极不合适的。涉及多服务多数据源的事务，应称为"分布式事务"。
>

全局事务基于[DTP模型](https://en.wikipedia.org/wiki/Distributed_transaction)实现，DTP( Distributed Transaction Processing Model)是由 X/Open 组织提出了一套名为[X/Open XA（XA 是 eXtended Architecture 的缩写）](https://zh.wikipedia.org/wiki/X/Open_XA)的处理事务架构。XA是一套与开发语言无关的通用规范，Java中定义了JSR 907 Java Transaction API，实现了全局事务处理的标准，就是我们现在所熟知的 JTA。

### **DTP 模型**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f5a20c21-c7c4-41e1-9857-cf92bdc59fa3/Untitled.png)

- AP(Application Program)：应用程序，主要是定义事务边界以及那些组成事务的特定于应用程序的操作。
- RM(Resouces Manager)：资源管理器，管理一些共享资源的自治域，如提供对诸如数据库之类的共享资源的访问。
- TM(Transaction Manager)：事务管理器，管理全局事务，协调事务的提交或者回滚，并协调故障恢复。

### **JTA 模型**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4ba6c719-469e-435b-b693-e5a9b9e80a94/Untitled.png)

JTA 定义了几个接口:

- javax.transaction.TransactionManager : 事务管理器，负责事务的begin, commit，rollback 等命令;
- javax.transaction.UserTransaction：用于声明一个分布式事务;
- javax.transaction.TransactionSynchronizationRegistry：事务同步注册;
- javax.transaction.xa.XAResource：定义RM提供给TM操作的接口;
- javax.transaction.xa.Xid：事务xid接口;

为了解决这个问题，XA 将事务提交拆分成为两阶段过程：

- Prepare 阶段: TM 向所有 RM 发送 prepare 指令，RM 接受到指令后，执行数据修改和日志记录等操作，然后返回可以提交或者不提交的消息给 TM。如果事务协调者 TM 收到所有参与者都准备好的消息，会通知所有的事务提交，然后进入第二阶段。
- Commit 阶段， TM 接受到所有 RM 的 prepare 结果，如果有 RM 返回是不可提交或者超时，那么向所有 RM 发送 Rollback 命令；如果所有 RM 都返回可以提交，那么向所有 RM 发送 Commit 命令，完成一次事务操作。

以上两阶段过程 就是[“两段式提交”（2 Phase Commit，2PC）协议](https://zh.wikipedia.org/wiki/%E4%BA%8C%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A4)，保证了多个数据源的ACID特性。

还是以 MySQL/InnoDB 为例，展开讲讲“两段式提交”的具体实现流程。

```sql
XA START "transfer_money"; -- 开启一个 XA 事务，后面的字符串是事务的 xid，这是一个唯一字符串，开启之后，事务的状态变为 ACTIVE
update account set amount=amount-10 where account_no='A'; -- 执行具体的 SQL
XA END "transfer_money"; -- 结束一个 XA 事务，此时事务的状态转为 IDLE
XA PREPARE "transfer_money"; -- 将事务置为 PREPARE 状态
XA COMMIT "transfer_money"; -- 提交事务，提交之后，事务的状态就是 COMMITED

-- 最后一步，可以通过 XA COMMIT 来提交，也可以通过 XA ROLLBACK 来回滚，回滚后事务的状态就是 ROLLBACK。
```

两段式提交协议原理简单，但有几个显著缺点：

- 单点问题: 协调者一旦发生宕机，将影响到所有参与者，都必须一直等待;
- 性能问题: 所有参与者是一个统一的整体，整个过程持续到其中最慢的参与者操作结束；期间有两次远程服务调用，三次数据持久化；
- 一致性风险: 网络分区场景下，当协调者发出提交事务且自己持久化完成，因网络断开，导致部分参与者无法提交事务，也无法回滚。

为了缓解两段式提交协议的一部分缺陷，具体地说是协调者的单点问题和准备阶段的性能问题，后续又发展出了[“三段式提交”（3 Phase Commit，3PC）协议](https://zh.wikipedia.org/wiki/%E4%B8%89%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A4)。

### **参考资料**

- 蚂蚁金服大规模分布式事务实践和开源介绍 <https://gw.alipayobjects.com/os/basement_prod/514cacb8-a7b9-4b39-b277-5e12ecb723d3.pdf>
- XA 事务水很深，小伙子我怕你把握不住！ <https://segmentfault.com/a/1190000040564227>
