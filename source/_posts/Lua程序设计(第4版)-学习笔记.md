---
title: Lua程序设计第4版-笔记
date: 2018-12-21 16:41:33
tags: lua
---

<center>Lua程序设计第4版-笔记<center>

# 语言基础

## Lua语言入门

### 三目运算

```lua
((a and b)or c))或(a and b or c)
--b不等于false时,等价于C语言的a?b:c
```



### 惯用写法 x=x or v

```lua
--等价于
if not x then x = v end
```

### type(nil) == nil 疑惑

返回是string类型，用nil做比较应用"nil"

```lua
> type(nil) == nil 
false
> type(nil) == "nil" 
true
```

### 编写打印自身名称的程序

```lua
print(arg[0])
--考点在编译器运行代码前会先创建arg表来存储命令行参数。
```

## 八皇后

### 输出1解

addqueen中添加os.exit()

```lua
--eight-queen.lua
N = 8 --棋盘大小
a = {3,7,2,1,8,6,5,4}
-- 打印棋盘
function printsolution(a)
	for i=1,N do
		for j=1,N do
			io.write(a[i]==j and "X" or "-"," ")
		end
		io.write("\n")
	end
end

-- 检查当前皇后是否会被攻击，c为列数
function isplaceok(a,n,c)
	for i=1,n-1 do
		if (a[i]==c) or --同一列
		   (a[i]-i == c-n) or	--右下对角线
		   (a[i]+i==c+n) then   --左下对角线
			return false --遭受攻击
		end
		
	end
	return true --不会被攻击
end

--添加皇后
function addqueen(a,n)
	if n > N then
		printsolution(a)
		os.exit()
	else
		for c=1,N do
			if isplaceok(a,n,c) then
				a[n] = c
				addqueen(a,n+1)
			end
		end
	end
end
addqueen({},1)
```

