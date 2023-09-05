---
title: mysql批量创建数据
date: 2023-09-05 11:48:13
tags:
---

# 创建表

```sql
CREATE TABLE employee2
(
    id        int primary key,
    LastName  varchar(255),
    FirstName varchar(255),
    Address   varchar(255),
    profile   varchar(255)
);
```



# 创建插入数据存储过程

```sql
CREATE PROCEDURE BulkInsert()
BEGIN
    DECLARE i INT DEFAULT 1;
    truncate table employee2;
    WHILE (i <= 20000) DO
            INSERT INTO employee2 (id,FirstName, Address) VALUES(i,CONCAT("user","-",i), CONCAT("address","-",i));
            SET i = i+1;
        END WHILE;
END
```

# 调用过程插入数据

```sql
CALL BulkInsert();
```

