# 网上图书管理系统

## 背景

IT 项目管理大作业，四人组队，以体验完整的项目管理过程为主要目的，成果为可运行程序和若干相关文档。

项目地址：

https://github.com/scuteam/Library_4/tree/dev

https://github.com/USER0308/Library_4/tree/dev

## 系统概述

系统使用者有：前台操作员，书籍操作员，内部操作员，读者和游客

其中，游客可以在搜索框输入书籍相关信息来搜索书籍

读者拥有帐号和密码，可以查看所有已借阅已归还的书籍和归还日期，当前已借阅尚未归还的书籍和待归还日期，对于未归还的书籍可选择续期

前台操作员负责为读者办理借阅或归还书籍业务，表现为：更新读者的借阅信息，更新图书馆书籍信息

书籍操作员负责图书上架和图书下架，表现为更新图书馆书籍信息

管理员负责管理系统内部人员信息，表现为：更新系统内部人员身份，增加或删除系统内部人员

## 技术架构

* 前端 Vue.js（javascript）

* 后端 Django（Python）

* 数据库 Sqlite3

## 系统设计

对游客而言，需要提供搜索框，搜索结果显示界面，和查看图书详细信息界面

对于所有读者，内部操作员，书籍操作员，前台操作员，都需要登录系统，在登录框中选择对应身份，跳转到不同界面

对于读者，需要维护已借阅已归还书籍信息板块和已借阅待归还书籍信息板块，在待归还表格中可对书籍进行续期操作

对于书籍操作员，需要维护图书上架和图书下架两大板块，在图书上架板块，填写书籍相关信息，上传书籍封面图片，在书籍下架板块，在搜索框输入书籍相关信息，搜索到待下架书籍，选择下架

对于前台操作员，需要维护办理借阅/归还书籍和使用读者身份证来为读者注册两大板块，在借阅板块，通过在输入框输入读者信息查询该读者名下所有待还书籍，可选择书籍进行还书，也可点击借书输入书籍信息进行借书

对于内部操作员，列出所有内部人员和身份（权限），可新增和删除内部人员，也可修改其身份（权限）;列出所有读者，可以新增或删除读者帐号

总而言之，系统需要搜索框，搜索结果界面，图书详情界面，两大板块的模板（根据登录者身份填充该模板）。

主页

![homepage](http://ovt2bylq8.bkt.clouddn.com/7ceca0a6b9b703983ebc0a58d94faa47.png)

<hr/>

查询结果

![query_result](http://ovt2bylq8.bkt.clouddn.com/a5f3e81bb2cc1671c57540ebef7e2690.png)

<hr/>

图书信息

![book_info](http://ovt2bylq8.bkt.clouddn.com/a91380c54191760ba2209cfab58c90e4.png)

<hr/>

登录

![login](http://ovt2bylq8.bkt.clouddn.com/6d6e16cc2223d96447a20f70a519a024.png)

<hr/>

读者查询待还书籍

![to_return_book](http://ovt2bylq8.bkt.clouddn.com/f03c4e565a2ba0a375a336eed0df275f.png)

<hr/>

读者查询已归还书籍

![returned](http://ovt2bylq8.bkt.clouddn.com/947d0091b916ba7e532b686ec1ef7989.png)

<hr/>

书籍操作员图书上架

![add-book](http://ovt2bylq8.bkt.clouddn.com/7c38214f8b5f75237633cc5f092817f1.png)

<hr/>

书籍操作员图书下架

![delete-book](http://ovt2bylq8.bkt.clouddn.com/c046d04cbe6c06174ca9d0bc5e0a98f3.png)

<hr/>

前台操作员借还书

![return-book](http://ovt2bylq8.bkt.clouddn.com/fa3f93152440adad0dce782c2a7b03bf.png)

<hr/>

前台操作员为读者注册帐号

![sign-in](http://ovt2bylq8.bkt.clouddn.com/caf077f5eef89e646ac118af45a90043.png)

<hr/>

管理员修改系统内部人员（操作员）角色（权限）

![add-inner-operationer](http://ovt2bylq8.bkt.clouddn.com/4778c70f75d1c1e99b7e77ecb3fa966a.png)

<hr/>

管理员新增内部人员（操作员）

![add-inner-operationer](http://ovt2bylq8.bkt.clouddn.com/235b92e090fb6ee594e2d3914d1c20dd.png)

<hr/>

管理员删除普通用户（读者）

![delete-common-people](http://ovt2bylq8.bkt.clouddn.com/66d4b5c19af15925a75b7bbae9438c46.png)

<hr/>

管理员新增普通用户（读者）

![add-common-people](http://ovt2bylq8.bkt.clouddn.com/826094692578887993b527031eb05133.png)


## 技术难点

### CSRF

采用前后端完全分离的方法意味着抛弃了 Django 的 MVT 框架，无法方便地使用 Django 在模板中的 {% CSRF_TOKEN %} 来防止 CSRF 攻击，采取的办法是绕过它，放弃 CSRF 检测，只需在 settings.py 的 MIDDLEWARE 中注释掉 django.middleware.csrf.CsrfViewMiddleware 即可


### 跨域问题

前后端完全分离，前端 Vue.js 有个服务器在监听 8080 端口，检测前端文件是否修改更新，生成 js 文件，前端请求后端时，请求发到的是 8080 端口,需要设置端口转发到后端监听的 8000 端口，这就存在着跨域问题

前端设置代理转发表
在 frontend/config/index.js 中 设置 proxyTable

```

proxyTable: {
  '/api': {
    target: 'http://127.0.0.1:8000/',
    changeOrigin: true
  }
},

```
请求后端时需要带上 ’/api‘ 前缀

后端服务器端安装 corsheaders

```

pip install django-cors-headers

```

并把 corsheaders 添加到 settings.py 中的 INSTALL_APPS 中，把 corsheaders.middleware.CorsMiddleware 添加到 settings.py 的 MIDDLEWARE 中且在 django.middleware.common.CommonMiddleware 之前，添加配置 CORS_ORIGIN_WHITELIST

```

CORS_ORIGIN_WHITELIST = (
    'localhost:8000',
    'localhost:8080',
    '127.0.0.1:8000',
    '127.0.0.1:8080'
)

```

参考链接： https://github.com/ottoyiu/django-cors-headers
除此以外，在编写 views 返回 response 给浏览器的时候，改写成

```

response = HttpResponse(json.dumps({'key': value}))
    response['access-Control-Allow-Origin'] = '127.0.0.1:8080'
    response['Access-Control-Allow-Credentials'] = 'true'
    return response

```

### 序列化与反序列化问题

JSON 有个 stringify，qs 也有 stringify，都是将对象字符串化，方便在网络中传输,两者区别

```

var a = {name:'hehe',age:10};
qs.stringify序列化结果如下
name=hehe&age=10

而JSON.stringify序列化结果如下：
"{"a":"hehe","age":10}"

```

特别是对象数组的时候更要小心使用两者

### Vue.js 中图片上传服务器问题

前后端分离使得 Django 的 ImageField 和 FileField 都无法正常使用，这就需要单独处理图片，把图片从内存中写入到磁盘，然后将图片与书籍关联起来

```

file = request.FILES.get('file')
    with open("static/image/%s" % file.name, 'wb+') as f:
        # 分块写入文件
        for chunk in file.chunks():
            f.write(chunk)
    print 'write over'

```

其实，个人觉得最困难的一步就在将数据库中的书籍信息与磁盘中的图片对应起来，因为在 Vue.js(ElementUI) 中采用上传图片的组件 <el-upload></el-upload> 指定了 action=‘special/path/’，然后根据 Django 的特性，就要为这个路径指定 views 的方法，而浏览器 POST 书籍的信息的时候也有路径和 views 方法，也就是说上传图片和上传基本信息是两个不同且单独的方法，难以保证两者之间的对应关系，书籍操作员可能只上传图片不上传图书基本信息，也可能只上传图书基本信息不上传图片。我采用的笨办法是：上传的某书本的封面图片文件名必须与该书籍的 ISBN 一致，提交书籍基本信息到服务器后会检查是否在图片目录下有以该书籍的 ISBN 开头的图片，有就将其对应关联起来，否则将图片指向“缺少封面”的默认图片。显示书籍时，在图片目录下找以对应书籍的 ISBN 开头的图片即可，同样地，找不到则以默认封面为准。这种办法既不方便又存在书籍操作员可能恶意上传图片浪费服务器磁盘的可能。

## 设计

MVCS 模式

models-views-controllers-services
