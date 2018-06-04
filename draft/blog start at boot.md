# gunicorn 开机启动
> 老早就搭建好博客了,django在上面也能运行得好好的,但是有个弊端,Nginx是开机启动的,但gunicorn不是开机启动的,所以每次服务器关机再开机访问网站时都会502Bad Gate.然而问题直到今天才着手去解决.

开始的解决思路就是弄个shell脚本,开机启动.shell脚本如下

```
nohup gunicorn blogproject:wsgi.application &
```

然后添加到/etc/rc.local中,内容如下

```
cd /path/to/blog/project
/bin/bash command.sh
exit 0
```
但是并不奏效.因为博客搭建好了快一年了,当时有些设置不太记得了,直接执行`cd /path/to/blog/project`和`/bin/bash command.sh`,发现不能启动,说明gunicorn指令出错了,于是在命令行执行`nohup gunicorn blogproject:wsgi.application &`发现还是无法启动,通过查看`/etc/nginx/site-enabled/default`,发现nginx对外监听80,对内监听8000端口,又想着会不会是没指定端口的原因,加上后在命令行执行`nohup gunicorn -b 127.0.0.1:8000 blogproject:wsgi.application &`,还是无法启动,通过查看nohup.out,发现找不到模块,原来是没激活虚拟环境,激活环境后终于可以在命令行中跑起来了.接下来就是添加到`/etc/rc.local`中,但是在指令都正确的情况下还是无法开机启动.

最后将之制作成服务,添加到开机启动.

创建/usr/lib/systemd/system目录

```
mkdir -p /usr/lib/systemd/system
```

创建/usr/lib/systemd/system/blog.service

```
vi /usr/lib/systemd/system/blog.service
```

内容如下:

```
[Unit]
After=syslog.target network.target remote-fs.target nss-lookup.target
[Service]
# 你的目录
WorkingDirectory=/path/to/blog/project
# gunicorn启动命令
ExecStart=/path/to/virtual/env/bin/gunicorn -b 127.0.0.1:8000 blogproject.wsgi:application
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

保存.

启动服务

```
sudo systemctl start blog
```

验证服务启动成功

```
ps -ef | grep gunicorn
```

添加服务到开机项

```
sudo systemctl enable blog.service
```
