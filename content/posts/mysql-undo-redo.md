+++
title = 'MySQL的Undo Log和Redo Log'
date = 2024-04-29T21:12:16+08:00
draft = false
tags = ["Mysql"]
+++

MySQL的InnoDB存储引擎使用undo log和redo log来保证事务的原子性、一致性和持久性。这两种日志在数据库的恢复和并发控制中扮演着关键角色。

## Undo Log（回滚日志）

### 定义
Undo Log是一种用于撤销事务对数据库所做更改的日志。

### 主要功能
1. **事务回滚**：在事务失败或显式回滚时，用于将数据恢复到事务开始前的状态。
2. **MVCC（多版本并发控制）**：用于实现读一致性，允许多个事务同时读写数据库。

### 工作原理
1. 当事务对数据进行修改时，InnoDB会先将原始数据保存到undo log中。
2. 如果事务需要回滚，系统会根据undo log中的信息将数据恢复到原始状态。
3. 对于读操作，InnoDB可以利用undo log来构造数据的历史版本，实现非锁定读。

### 特点
- 存储在系统表空间或独立的undo表空间中。
- 是逻辑日志，记录的是对数据的逻辑操作。
- 在事务提交后不会立即删除，用于MVCC。

## Redo Log（重做日志）

### 定义
Redo Log是用于记录事务对数据页的修改的物理日志。

### 主要功能
1. **确保事务的持久性**：即使在系统崩溃的情况下，已提交的事务修改也不会丢失。
2. **提高性能**：通过先写日志再修改数据页的方式（WAL，Write-Ahead Logging），减少随机IO，提高性能。

### 工作原理
1. 当事务修改数据时，修改会先写入redo log buffer。
2. 根据不同的配置，redo log会以不同的频率刷新到磁盘。
3. 在系统崩溃后重启时，InnoDB会使用redo log来恢复未写入数据文件的已提交事务修改。

### 特点
- 是物理日志，记录的是对于数据页的修改。
- 是循环写入的，空间固定。
- 通常有两个文件组成，交替使用。

## Undo Log和Redo Log的区别

| 特性 | Undo Log | Redo Log |
|------|----------|----------|
| 主要目的 | 事务回滚和MVCC | 确保持久性和提高性能 |
| 日志类型 | 逻辑日志 | 物理日志 |
| 记录内容 | 数据修改的逆操作 | 数据页的实际修改 |
| 使用时机 | 事务回滚、读取历史版本 | 系统崩溃恢复 |
| 删除时机 | 在不再需要时（如旧版本不再需要） | 循环覆盖 |

## 总结

1. Undo Log主要用于事务的回滚和实现MVCC，保证了事务的原子性。
2. Redo Log主要用于保证事务的持久性，同时通过WAL机制提高了数据库的性能。
3. 这两种日志共同工作，确保了MySQL在各种情况下（包括事务回滚和系统崩溃）都能保持数据的一致性和完整性。