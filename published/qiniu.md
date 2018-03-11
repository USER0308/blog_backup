# Atom 搭配七牛图床, 像 word 一样 Ctrl + v 粘贴图片, 上传到图床

到七牛官网注册帐号, 创建存储空间, 获得分配的域名, 获得密钥 AK/SK 对

![domain](http://ovt2bylq8.bkt.clouddn.com/45db02a83c0cb316627838eb5a981adb.png)

![AK/SK](http://ovt2bylq8.bkt.clouddn.com/03374b8121958e72641a835f75b2cfaf.png)

下载 Atom, 下载插件 markdown-assistant 和 qiniu-uploader, 其中 qiniu-uploader 可能下载失败, 多试几次.

在 qiniu-uploader settings 中配置 AK,存储空间名,域名,SK,编写 markdown 时只需 CTRL + V 粘贴图片即可上传到存储空间.
