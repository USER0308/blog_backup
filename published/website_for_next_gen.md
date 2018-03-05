# 项目后期报告

## 概述：

### 1.1 项目背景

(1): 13 届师兄提出想要建个 IBM 内部网站

(2): 14 届师兄完成网站首页，关于，申请加入等功能，用于 16 届招新用

(3): 15 届师兄，也就是我，参加数据库大作业和数据库实训时，挑取了论坛作为选题，一方面扩充网站功能，一方面巩固之前完成的自写博客系统用到的技术，一方面锻炼自己的编程能力

(4): 希望以后的师弟师妹们可以继续完善我们的网站，为俱乐部贡献自己的一份力量

其他：

  项目名称：IBM BBS

  项目概要设计：采用 B/S 架构，用户通过浏览器访问 BBS 网站，网站提供功能：游客填写申请表 (招新用), 游客使用邀请码注册成为会员，会员登录进入论坛，会员签到获取积分，会员修改个人信息，会员在指定板块上发表帖子，会员浏览不同板块的帖子，会员回复帖子，会员回复会员，会员上传文件获得积分，会员消耗积分下载资料，会员查看会员信息，会员与会员私信，管理员登录后台，管理员查看游客填写的申请表，管理员对游客发送邀请码，管理员管理会员，管理员管理帖子，管理员管理回帖，管理员管理资源

  系统总体功能图：

  ![design](http://ovt2bylq8.bkt.clouddn.com/a51a4a0a0707a0e36e9ec1847c44fca8.png)

  项目技术栈：前端 html css js(引用 bootstrap 库美化界面，jquery 库添加动态效果)，后端 Python(采用 Django 框架方便管理后台)

  数据库 E-R 模型图：

   ![database design](http://ovt2bylq8.bkt.clouddn.com/e74fd20e7bffde49282906a38036545c.png)

## 业务功能模块

* ~~ 游客填写申请表~~
* 游客使用邀请码注册成为会员 (已完成，待美化)
* ~~ 会员登录进入论坛~~
* 会员签到获取积分 (已完成)
* 会员修改个人信息 (已完成，待美化)
* 会员在指定板块上发表帖子 (已完成)
* 会员浏览不同板块的帖子 (已完成，待美化)
* 会员回复帖子 (已完成)
* 会员回复会员 (已完成)
* 会员上传文件获得积分 (已完成，待美化)
* 会员消耗积分下载资料 (已完成，有 bug)
* 会员查看会员信息 (已完成，待美化)
* 会员与会员私信 (半完成，有 bug)
* ~~ 管理员登录后台~~
* ~~ 管理员查看游客填写的申请表~~
* 管理员对游客发送邀请码 (已完成，有 bug)
* ~~ 管理员管理会员~~
* ~~ 管理员管理帖子~~
* ~~ 管理员管理回帖~~
* ~~ 管理员管理资源~~

\* 注：

游客填写申请表部分与会员登陆进入论坛部分模块 14 级师兄已经实现

管理员登录后台，查看游客填写的申请表，管理会员、帖子、回帖、资源部分模块 Django 框架的 admin 后台已经实现

游客使用邀请码注册成为会员、会员修改个人信息、会员查看会员信息模块界面需要美化

会员消耗积分下载资料、会员与会员私信、管理员对游客发送邀请码模块存在 bug

### 2.1 模块列表

前端完成情况：基本功能已完成，需要 css、js 美化，添加动态效果
后端完成情况：基本功能已完成，需要重建 model，删掉部分没用到的属性，重新构建 model 之间的对应关系
与数据库交互：正常

示例：

2.1.1 游客使用邀请码注册模块前端
界面展示：
![sign_up](http://ovt2bylq8.bkt.clouddn.com/5096609e493d27d1f29839b16b4ee134.png)

源码位置：

  html 模板位置：IBMsite/templates/sign_up_templates.html

业务流程图

  用户角度：用户打开 URL-- 填写注册表单 -- 点击提交 -- 提示成功

  系统角度：Django 获得 URL -- 根据 URL 匹配原则调用相应的 view 函数 -- 返回一张空的表格给用户 -- (用户提交表单后) Django 获得 URL 和用户数据 -- 根据 URL 匹配原则调用相应的 view 函数 -- 返回成功界面给用户

开发速度：基本功能已实现，待美化

期望值
![join](http://ovt2bylq8.bkt.clouddn.com/13f8cd911b8c9aa68c928acf8fb55be5.png)

2.1.2 游客使用邀请码注册模块后端

源码位置：

  传参数给模板的 view 位置：IBMsite/mysite/views/sign_up_views.py,

  view 中使用到的 Form 位置：IBMsite/mysite/forms.py，

  ~~form 中对应的 model 位置：IBMsite/mysite/models/~~

关联数据库：本模块无

开发速度：基本功能已实现

其他模块类似

### 数据库依赖关系

数据库表格: 通过 model 来定义，一个 model 就是一个数据库表格

  * application_models(申请表)

  * chat_models(私信)

  * code_models(邀请码)

  * comment_models(评论)

  * manager_models(高层管理人员)

  * member_models(会员)

  * member_upload_models(会员与上传的文件相关)

  * post_models(帖子)

  * sign_models(签到与积分相关)

表格之间的依赖关系

  * application_models 独立

  * member_models 独立

  * manager_models 依赖于 member_models, 先要成为 member_models 才能成为 manager_models

  * code_models 依赖于 manager_models, 先要有 manager_models 才能有 code_models

  * post_models 依赖于 member_models, 先要有 member_models 才能有 post_models

  * comment_models 依赖于 post_models 和 comment_models

  * chat_models 依赖于 member_models

  * member_upload_models 依赖于 member_models

  * sign_models 依赖于 member_models

故而执行 python ./manage.py runserver 之后通过 http://127.0.0.1:8000/admin 访问后台 admin 后，应该先在 member_models 中创建一个 member 对象，然后在 manager_models 中将 manager 对象指向该 member，position 填 admin，然后回到 http://127.0.0.1:8000/mysite/application/ 按照照常顺序填写申请表，回到 admin 后台查看申请表，发送邮件，查看邀请码，通过邀请码注册，使用帐号和密码登录 (登录时可能有 bug，没能成功跳转到 home_page 页面)

## bug 列表

  * 会员登录：有时无法跳转到 home_page 界面

  * 会员消耗积分下载：下载文件功能尚未实现，目前只是将 URL 指向了别的网站的一个可下载的文件，提示积分不足后，仍会继续执行下载功能

  * 会员与会员私信：

  1、没有打开与该会员的对话框，示例：A 查看 B 的资料后点击私信，会跳转到 A 的所有未读私信，而不是与 B 的私信对话框

  2、回车发送消息后，消息没有写入数据库

  3、阅读了私信后，没有将信息标记为已读

  * 管理员对游客发送邀请码：部署到服务器后关掉连接服务器的终端或者终端会话过期后无法发送邮件

  * 点击某一板块时应该将该板块background-color加上

  * 发帖界面应将资源分享区移去

## 未来展望

把板块作为一个类,
加入富文本编辑器
添加表情
