# INSERT INTO ... SELECT解析

## 资源

- [Innodb:RR隔离级别下insert...select 对select表加锁模型和死锁案列](https://www.jianshu.com/p/f08bf6f67410)
- [MySQL 中隔离级别 RC 与 RR 的区别](https://www.cnblogs.com/softidea/p/9185206.html)

## Background

业务中的快照 与 MySQL中的快照(一致性读, Read View)不同:

- MySQL快照是逐行判断可见性的, 业务中的快照的语义与锁表后读取的含义一致.


### MySQL 5.7参考手册中的阐述

```
14.7.2.3 Consistent Nonlocking Reads

The type of read varies for selects in clauses like INSERT INTO ... SELECT, UPDATE ... (SELECT), and CREATE TABLE ... SELECT that do not specify FOR UPDATE or LOCK IN SHARE MODE:

By default, InnoDB uses stronger locks in those statements and the SELECT part acts like READ COMMITTED, where each consistent read, even within the same transaction, sets and reads its own fresh snapshot.

To perform a nonlocking read in such cases, enable the innodb_locks_unsafe_for_binlog option and set the isolation level of the transaction to READ UNCOMMITTED, READ COMMITTED, or REPEATABLE READ to avoid setting locks on rows read from the selected table.



14.7.3 Locks Set by Different SQL Statements in InnoDB

INSERT INTO T SELECT ... FROM S WHERE ... sets an exclusive index record lock (without a gap lock) on each row inserted into T. If the transaction isolation level is READ COMMITTED, or innodb_locks_unsafe_for_binlog is enabled and the transaction isolation level is not SERIALIZABLE, InnoDB does the search on S as a consistent read (no locks). Otherwise, InnoDB sets shared next-key locks on rows from S. InnoDB has to set locks in the latter case: During roll-forward recovery using a statement-based binary log, every SQL statement must be executed in exactly the same way it was done originally.
```


### RR下死锁场景

- Innodb:RR隔离级别下insert...select 对select表加锁模型和死锁案列: https://www.jianshu.com/p/f08bf6f67410

```
              trx1                                      trx2
--------------------------------------|--------------------------------------
BEGIN                                 | BEGIN
--------------------------------------|--------------------------------------
UPDATE b SET col = 1 WHERE id = 2999; |
(LOCK_X 2999)                         |
--------------------------------------|--------------------------------------
                                      | INSERT INTO a SELECT * FROM b WHERE id in (999, ..., 2999);
                                      | (LOCK_S 999)
UPDATE b SET col = 1 WHERE id = 2999; |
(LOCK_X 999) WAIT                     |
                                      | (LOCK_S 2999) WAIT
--------------------------------------|--------------------------------------
```

### RC下数据不一致场景


```
SELECT id, col FROM b WHERE id in (999, 2999);

----|----
id  | col
----|----
999 | 100
----|----
2999| 100
----|----
```

```
              trx1                                    trx2      
--------------------------------------|--------------------------------------
BEGIN                                 | BEGIN
--------------------------------------|--------------------------------------
                                      | INSERT INTO a SELECT * FROM b WHERE id in (999, ..., 2999);
--------------------------------------|--------------------------------------
                                      | (READ 999, OUT: 100)
--------------------------------------|--------------------------------------
UPDATE b SET col = col-10             |
WHERE id = 999;                       |
--------------------------------------|--------------------------------------
UPDATE b SET col = col+10             |
WHERE id = 2999;                      |
--------------------------------------|--------------------------------------
COMMIT                                |
--------------------------------------|--------------------------------------
                                      | (READ 2999, OUT: 110)
--------------------------------------|--------------------------------------
```

## 代码走读

TODO(zhoujiagen)
