---
layout: post
title: mysql开启日志,删除重复记录只保留一条
data: 2018-09-28
categories: blog
tags: [mysql,log]
description: 
---

# mysql开启日志记录功能

## 查看mysql日志是否开启，日志保存地址
```javascript 
show variables like '%general_log%'; 
```

## 查看日志输出格式、地址
```javascript 
show variables like 'log_output';
```

## 开启mysql日志功能
```javascript 
set global general_log = on;
```

## 关闭mysql日志功能
```javascript 
set global general_log = off;
```

## 删除mysql多字段重复记录，只保留一条
```javacript
DELETE
FROM
    sys_app_real_meter_bill
WHERE
(tenant_id, app_code) IN (SELECT
            c.tenant_id,
            c.app_code
				from (
        SELECT
            tenant_id,
            app_code
        FROM
            sys_app_real_meter_bill
        GROUP BY
            tenant_id,
            app_code
        HAVING
            count(*) > 1
    ) c)

 AND    app_real_meter_bill_id NOT IN (
select b.id from (
    SELECT
        min(app_real_meter_bill_id) id
    FROM
        sys_app_real_meter_bill
    GROUP BY
        tenant_id,app_code
    HAVING
        count(*) > 1
) b);
```
