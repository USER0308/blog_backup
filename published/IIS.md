# IIS配置

## 总览:

![](http://ovt2bylq8.bkt.clouddn.com/830946bd400bba3f9a14f438b9045f0e.png)

## MIME 类型

![mime](http://ovt2bylq8.bkt.clouddn.com/7b68bac602916e7dbd5abf298245333c.png)

HTTP 请求的头部有个字段叫 Content-Type, 用于指定文件类型, 比如 Content-Type=application/octet-stream 表示可供下载的二进制文件, Content-Type=text/xml 表示纯文本文档, 可以直接在浏览器打开, text/css 表示样式.

MIME 用于将文件后缀名和文件类型关联起来, 假如将 .png 的关联类型由 application/x-png 改为 application/octet-stream 时, http 请求访问该资源时,则设置头部 Content-Type 为指定的 application/octet-stream, 浏览器按照 application/octet-stream 解析该 png, 把它下载保存为本地文件而不是在浏览器网页中展示

要显示(准确来说是,告知浏览器怎么解析)某些奇怪后缀名的文件,记得在这里加上后缀名和相应解析的MIME 类型

## 默认文档

![default](http://ovt2bylq8.bkt.clouddn.com/f79c1d7d3e9e836413045d5f2a1838be.png)

当客户端未请求特定文件名时,指定返回的默认文件

一个 Http 请求的 URL 长这样子: http://hostname.com/path/to/index.html, http 代表 http 的请求协议, hostname.com 代表域名, path 代表一级目录, to 代表二级目录,file.html 代表请求的文件,有时候输入 http://hostname.com/path/to/ 不指定特定文件名时也能返回 http://hostname.com/path/to/index.html 这个文件,因为在默认文档里指定了默认返回文件里包含了 index.html

所以,输入http://hostname.com/ 实际访问的是http://hostname.com/index.html

如果网站根目录下只有一个 main.html, 把 main.html 加入默认文档里,这样,访问 http://hostname.com/ 实际返回的是 main.html

## 目录浏览

假设网站根目录下既没有 main.html, 也没有 index.html, 默认文档里列出的都没有,那这样的 http 实际是请求一个目录,如果开启了目录浏览就会返回请求的这个目录下所有子目录和文件,关闭了目录浏览就会网页报错

## 身份验证

指定谁可以访问,谁不能访问网站,启用匿名身份验证表示谁都可以访问网站内容

## 其他

### 编辑权限

![privillage](http://ovt2bylq8.bkt.clouddn.com/890eb7d29ff86661e45bec26bac614a3.png)

IIS 用户组必须拥有网站根目录的文件夹(C:\\ado)的读写权限

打开文件夹的属性-安全-编辑-添加-高级-立即查找,找到 IIS 的用户组,一般名字叫 IIS_IUSRS, 确定-确定,编辑 IIS_IUSES 的权限,完全控制就省事了

### 绑定

类型：http

主机名不写

端口：80

IP地址：*

### 基本设置

物理路径指定网站根目录

连接为.. 特定用户,设置用户名,密码.一般来说会在系统中新增一个专门的用户用于此处,防止该用户权限过大,这里直接用了 administrator, 这是不恰当的

### 最后

网站根目录下要放什么

```
C\:ado

|--index.html

|--html\

|--css\

|--js\

|--others
```


相对路径的访问

* index.html 访问 css js 通过 ./css 和 ./css 访问

* html 里的 html 通过 ../css 和../js 访问

* index.html 访问 html 里的 .html 通过 ./html/XX.html 访问

* html 里的 .html 访问 index.html 通过 ../index.html 访问
