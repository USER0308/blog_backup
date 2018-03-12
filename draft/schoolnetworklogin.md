# 校园网脚本登录

> 原来项目地址: https://github.com/USER0308/schoolNetClientApp
> 在上面的项目中, 通过安卓的 webview 打开指定网址, 再执行 js 脚本代码, 自动填充表单, 人为地点击提交按钮, 在此过程中, 只是省略了手动输入帐号和密码信息, 并没有涉及到背后登录的原理, 下面就背后登录原理来进行分析并编写 python 脚本实现一键登录

首先, 连上校园网 wifi, 打开任意网站, 由于没有登录, 会自动重定向到登录界面, 得到登录界面的 URL 为
`https://s.XXXX.edu.cn/a70.htm?wlanuserip=192.168.165.193&wlanacip=192.168.255.251&wlanacname=WX6108E-slot5-AC&redirect=&session=&vlanid=XXXX-student&ip=192.168.165.193&mac=000000000000`
(上述 URL 已省去敏感信息)
F12(chrome) 打开开发者工具, 刷新页面, 抓到数据包, 主要包括一个 a70.htm 页面和一个功能函数 js 脚本, 还包括一些美化的 js,css, 在登录界面提交按钮出右键, 检查元素, 定位到按钮的 html 源代码,
``
只有 class 和 id, 得到 button 的 id 为
在抓包列表中打开功能函数 js 脚本, 将其全选, 复制粘贴到本地, 大概地浏览了一下, 主要分为几部分, 开头定义了一些全局变量, 主要用到的且比较重要的全局变量有
```
var accountSuffix="";// 账号后缀

var enPortal=1;// 是否支持 Portal 协议，进行第三方 AC 认证 (0 - 不支持；1 - 支持)

var enPerceive=0;// 是否支持快速登录 (0 - 不支持；1 - 支持)

var autoPerceive=0;// 快速登录是否允许自提交 (0 - 显示快速登陆；1 - 直接无感知)

var enHttps=1;// 是否需要 Https(0 - 不需要；1 - 需要)

var enMd5=0;// 是否需要 MD5(0 - 不需要；1 - 需要)var accountSuffix="";// 账号后缀

var enPortal=1;// 是否支持 Portal 协议，进行第三方 AC 认证 (0 - 不支持；1 - 支持)

var enPerceive=0;// 是否支持快速登录 (0 - 不支持；1 - 支持)

var autoPerceive=0;// 快速登录是否允许自提交 (0 - 显示快速登陆；1 - 直接无感知)

var enHttps=1;// 是否需要 Https(0 - 不需要；1 - 需要)

var enMd5=0;// 是否需要 MD5(0 - 不需要；1 - 需要)
```
在上面的变量中可以看到, 帐号后缀为空, 不需要 MD5 进行加密
继续看下面的代码, 花费了颇多的代码设置广告页面广告内容广告链接, 但都是没有派上用场, 然后又写了一些公有函数, 主要是正则匹配查找, 替换, 计算的, 再接着是一些奇怪的函数, 名字都是 aa,bb,cc 这些, 有进制间转换, 还有个 MD5 核心函数, 内容大概和加密有关, 接着到了最重要的登录验证函数, 函数如下:
```
// 登录认证事件 (form_id 为自定义表单 id: 1 - 表单 f1;2 - 表单 f2;3 - 表单 f3)
function ee(form_id){
	if(form_id == 1){// 会员认证
		document.getElementById("login").disabled = true;

		/** 验证会员登录表单 */
		// 省略, 防止出现表单为空

		/** 设置会员数据到隐藏表单 */
		document.f0.DDDDD.value=document.f1.DDDDD.value + accountSuffix;// 增加账号后缀
		if(enMd5 == 1){// 支持 MD5
			// 进行 MD5 加密
		}
		else{
//			document.f0.upass.value=xproc1(document.f1.upass.value);
			document.f0.upass.value=document.f1.upass.value;
			document.f0.R2.value="";
		}

		if(typeof(document.getElementsByName("save_me")[0]) == "object"){
			if(document.getElementsByName("save_me")[0].checked){
				var uname = document.f1.DDDDD.value;
				var pass = document.f1.upass.value;
				setCookie("md5_login",uname+"|"+pass);
			}
			else {
				delCookie("md5_login");
			}
		}

		document.getElementById("login").disabled=false;
	}
	else if(form_id == 2){// 手机认证
		// 省略
	}
	else{// 二维码认证
		// 省略
	}

	if(enPortal == 1){// 支持 Portal 协议, 进行第三方 AC 认证
		var vlan = "";
		if(getQueryString('vlanid') != null && getQueryString('vlanid') != ''){
			vlan = getQueryString('vlanid');
		}
		else{
			vlan = vlanid;
		}

		if(enHttps == 1){// 需要 Https(需要 EPOrtal 另置接口支持)
			document.f0.action = "https://" + window.location.hostname + ":801/eportal/?c=ACSetting&a=Login&wlanuserip="
								+ getQueryString('wlanuserip') +"&wlanacip="+ getQueryString('wlanacip')
								+ "&wlanacname="+ getQueryString('wlanacname') +"&redirect="+ getQueryString('redirect')
								+ "&session="+ getQueryString('session') +"&vlanid="+ vlan
								+ "&port="+ window.location.port +"&iTermType="+ getTermType()
								+"&protocol=https:";
		}
		else{
			// 省略
		}
	}
	else{// 本地认证
		// 省略
	}

	document.f0.submit();
	return false;
}
```
对这段代码进行提取, 主要是, 获得 f1 表单中的内容, 然后填写到隐藏表单 f0 中, 当然, 中间可能有加密步骤, 但由于全局变量 enMd5 = 0, 所以并没有调用到加密函数, 现在回到 html 页面中查看 f1 和 f0, 代码如下:
```
```
可以看到 f1 嵌套在 f0 中, f0 表单主体有
```
'DDDDD':$ACCOUNT,
'upass':$PASSWORD,
'R1':'0',
'R2':'',
'R6':'0',
'para':'00',
'0MKKey':'123456'
```
然后回到 js 脚本中, 查看哪些地方修改了 DDDDD,upass,R1,R2,R6,para,0MKKey 这些值, 通过 `ctrl + F` 快速搜索, 发现 R1 代表是否使用了 MD5 加密, R2 代表添加的帐号后缀, R6,para 和 0MKKey 都没有变动. 于是根据以上信息, 我们可以构造出一个表单, 然后提交给服务器.
先使用 Postman 提交测试一下,
![postman](http://ovt2bylq8.bkt.clouddn.com/d7c7eb0d5cc0d1346eda0eb6211ba429.png)
成功!
然后就可以使用 Python 编写提交表单的代码了, 过程省略不说.
再继续说说 js 脚本剩下的内容, 接下来就是处理注销的函数, 可以省略, 还有动态播放广告的函数, base64 编码和解码的函数, Unicode,UTF-8,UTF-16, 汉字之间转换函数, 最后就是报错信息的分类, 判断和提示, 至此, js 脚本分析完毕.

在测试过程中发现, 表单提交之后会跳转到指定的 ip 地址, 甚至连域名都没有, 如果登录成功的话, 会跳转到 3.htm 页面, 失败的话会跳转到 2.htm, 这两个页面的 `<title></title>` 不一样, 可以作出登录结果判断.
以下为 Python 登录脚本:
```
# encoding=utf-8
# /usr/bin/python
import urllib2,urllib
import re

url = "https://s.XXXX.edu.cn:801/eportal/?c=ACSetting&a=Login&wlanuserip=192.168.165.193&wlanacip=192.168.255.251&wlanacname=WX6108E-slot5-AC&redirect=&session=&vlanid=XXXX-student&port=80&iTermType=&protocol=https:"

body = urllib.urlencode([('DDDDD','account'),
  ('upass','password',
  ('R1','0'),
  ('R2',''),
  ('R6','0'),
  ('para','00'),
  ('0MKKey','123456')])

req = urllib2.Request (url,body)
response = urllib2.urlopen(req)
html = response.read().decode('gb2312')
# print html
p = r"<title>(.+?)</title>"
macher = re.findall(p,html)
title = ''.join(macher)
if title == u'Drcom PC 登陆信息页':
	print 'error'
elif title == u'Drcom PC 登陆成功页':
	print 'success'

```
当然,中间少不了正则匹配,转码等操作
