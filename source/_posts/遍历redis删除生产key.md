---
title: 遍历redis删除生产key
date: 2023-07-03 09:54:40
tags:
---



某天运维说，你的系统redis存在大量的key不设置过期时间，而且key是乱码的（其实是jdk序列化的key）。

我让运维备份出来，连上去，使用scan 0 match * count 10000，查看发现大量的key是:b'\xac\xed\x00\x05t\x00\x1a'开头的，以下是删除脚本



```python
import redis

# Redis 
redis_host = 'xxx.xxx.xxxx'
redis_port = 6379
redis_db = 2

# 分别扫描这三种模式的key删除
key_pattern = "hello*" 



r = redis.Redis(host=redis_host, port=redis_port, db=redis_db)

batch_size = 100
cursor = 0
ct = 0

while True:

    cursor, keys = r.scan(cursor=cursor, match=key_pattern, count=batch_size)

    for key in keys:
    	r.delete(key)
    	ct = ct+1
    if cursor == 0:
        break
print(ct)
print('finsh')

```

