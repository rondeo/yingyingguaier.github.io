---
title: Lua程序设计第4版-笔记
date: 2018-12-21 16:41:33
tags: lua
---

# Lua程序设计第4版-笔记


## Lua语言入门

### 三目运算

```lua
((a and b)or c)或(a and b or c)
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

## 强制返回一个结果

```lua
--使用括号强制返回第一个返回值
function foo2() return "a", "b" end
```

![001](/img/lua/001.png)

## table.unpack

```lua
--将列表转化成一组返回值
table.unpack({"Sun","Mon","Tue","Wed"},2,3)

--结果
Mon	Tue
```

## 文件操作

### 简单I/O模型

```lua
io.read("a") --读取整个文件
io.read("l") --读取下一行（丢弃换行符）
io.read("L") --读取下一行（保留换行符）
io.read("n") --读取一个数值
io.read(num) --以字符串读取num个字符

-- 只读打开指定文件，并将文件设置为当前输入流
io.input([file])

-- 设置默认输出文件
io.output([file])

-- 不带参数关闭默认文件
io.close([file])

-- 不带参数使用默认文件，返回一个迭代器，用在for循环
io.lines([file])

-- 将字符串或数字写入输出流
io.write(value)

-- 将换缓冲数据写入文件
io.flush()
```

### 完整I/O模型

```lua
-- 检查错误方法
local f = assert(io.open(filename,mode))
```

seek获取设置文件位置

```lua
f:seek(whence,offset)
-- 返回当前位置1相对于文件开头的偏移
-- whence指定使用偏移的字符串  		
-- set文件头 cur当前位置 end文件尾部 
-- offset 偏移量
```

### 运行系统命令

```lua
os.execute 和 io.popen
```

