---
title: mysql知识
date: 2020-01-03 09:51:57
tags:
---
## 批量插入测试数据

```sql
DROP PROCEDURE
    IF EXISTS test_insert;

DELIMITER ;;
CREATE PROCEDURE test_insert()

BEGIN
    DECLARE number INT DEFAULT 1;
    WHILE (number <= 10000) DO

        insert into testabc(a,b,c) values(concat('a',number),concat('b',number),concat('c',number));
        SET number = number + 1;
        END WHILE;
    commit;
END;;
CALL test_insert();
```

## mysql日志

### binlog

binlog是在**server层的**两个主要的用途

1. 主从复制
2. 数据恢复

binlog的格式也有3种

1. statement：基于sql，数据有可能丢失，像now()函数
2. row:文件大，数据不丢失
3. Mixed

Undolog

## icp索引下推

加上有联合索引a、b、c字段，where a =

# 