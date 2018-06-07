# 准备工作

## 创建虚拟机

虚拟机的网络模式 NAT

选择语言 - 中文

![](http://ovt2bylq8.bkt.clouddn.com/a51f78b91a6fd1dbfbb288c1fdffc889.png)

软件选择

![](http://ovt2bylq8.bkt.clouddn.com/a8c017f935b1c42308c5ba678432c03a.png)

选择带 GUI 的服务器

![](http://ovt2bylq8.bkt.clouddn.com/a7558c2c9c982e0528e723728dc87eda.png)

系统 - 网络和主机名

![](http://ovt2bylq8.bkt.clouddn.com/3c439ea233e147b8703944cd14ec49e7.png)

开启网络

![](http://ovt2bylq8.bkt.clouddn.com/03ff58e179ffd30d950fa1d0ea3ac40b.png)

设置 root 密码和用户名, 密码

![](http://ovt2bylq8.bkt.clouddn.com/2233943479aa35747f3d7a514e068741.png)

重启, 登录, 打开终端, 切换到 root

`su`

(输入 root 密码)

以下命令皆为 root 状态下输入

## 把所需资料复制进虚拟机中

创建 /fisco-bcos 文件夹

`mkdir /fisco-bcos`

宿主机为 linux:

linux 下载安装 ssh 服务, `sudo apt install openssh-server`
在虚拟机中通过 scp 命令远程复制宿主机目录下的文件

`*scp -r username@110.11.11.11:/home/username/file/ /fisco-bcos/*`

username 为宿主机的用户名, 110.11 为宿主机的 ip 地址, file 目录下有
* *163.repo*
* *docker-ce-17.12.0.ce-1.el7.centos.x86_64.rpm*
* FISCO-BCOS.tar.gz
* fisco_laste_image.tar
* nodejs.tar.gz
* script.tar.gz
* tool.tar.gz

确保把以上文件复制到 /fisco-bcos 文件夹

备注: 若宿主机为 Windows, 可选方案: 虚拟机关机, 网络设置为桥接, 重新开虚拟机, 更改虚拟机的 IP 配置:

`vi etc/sysconfig/network-scripts/ifcfg-enp0s3`

![](http://ovt2bylq8.bkt.clouddn.com/6205ba7bd773fe0e6c791728c320b86a.png)

ip 地址, 网关, dns 根据场景设置, 确保宿主机能 ping 通虚拟机, 然后用 Xshell 连接, 传文件, 这里不多说, 理论上是可行的, 实际尚未尝试

## 更新 yum 源

```
cd /fisco-bcos

mv 163.repo /etc/yum.repos.d/

yum makecache

yum install -y lrzsz wget net-tools

firewall-cmd --zone=public --add-port=53300/tcp --add-port=52300/tcp --add-port=35500/tcp --permanent

firewall-cmd --reload
```

## 安装 docker

```
cd /fisco-bcos

yum install -y container-selinux.noarch libcgroup.x86_64 libseccomp.x86_64 libtool-ltdl.x86_64

rpm -i docker-ce-17.12.0.ce-1.el7.centos.x86_64.rpm
```

## 启动 docker

`service docker start`

把 docker 添加到开机启动

`systemctl enable docker`

## 导入镜像

`docker load -i fisco_latest_image.tar`

## 安装智能合约编译器

```
cd /fisco-bcos
tar -zxf tools.tar.gz
cp ./tools/fisco-solc  /usr/bin/fisco-solc
chmod +x /usr/bin/fisco-solc
```

## 安装 docker-compose

```
cd /fisco-bcos
cp ./tools/docker-compose  /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## Nodejs 安装

```
cd /fisco-bcos
mv nodejs.tar.gz /home/
cd /home  # 注意
tar -zxf nodejs.tar.gz
ln -s /home/node-v8.9.4-linux-x64/bin/node /usr/local/bin/node
ln -s /home/node-v8.9.4-linux-x64/bin/npm /usr/local/bin/npm
ln -s /home/node-v8.9.4-linux-x64/bin/cnpm /usr/local/bin/cnpm
ln -s /home/node-v8.9.4-linux-x64/bin/babel-node /usr/local/bin/babel-node
```

# 区块链系统部署

## 初始化目录

```
mkdir -p /bcos-data  /bcos-log
cd /fisco-bcos
tar -zxf FISCO-BCOS.tar.gz
tar -zxf scripts.tar.gz
```

`cd /fisco-bcos/scripts`

查看虚拟机 ip

`ip addr`

一般为第二个, 假设为 10.0.2.15

## 创建并启动创世节点

`sh run_genesis.sh 10.0.2.15`

1+7+5 个回车

## 创建普通节点配置

`sh generate_config.sh 1 /bcos-data/node0 10.0.2.15`

修改端口, 上一条命令参数节点 id 为几, 端口号就相应加上几

`vi /bcos-data/node1/config.json`

```
rpcport=35501
p2pport=53301
channelport=52301
倒数几行的 node1 处还要改成 port=53301
```

`vi /bcos-data/node1/node.json`

端口号 +

## 注册普通节点

`sh register.sh node1`

## 启动普通节点

`sh start_container.sh /bcos-data/node1 /bcos-log`

## 检查共识

`tail -f /bcos-log/node0/* | grep ++++`

# 其他:

`docker ps`

列出所有正在运行的容器

`docker ps -a`

列出所有容器

容器前面一串十六进制字符是容器 id, 可以只取前四位来标识一个容器
停止容器运行

`docker stop a755`

删除容器

`docker rm -f a755`

清楚区块链系统产生的数据和日志

`rm -rf /bcos-data/`

`rm -rf /bcos-log/`

下次再次开机时直接启动容器即可, 不必再次部署环境

`docker start a755`

`docker start 63d2`

可查看达成共识

确保 docker 已经在运行, 若未运行, 则执行 `(sudo) service docker start`
或将docker添加到开机启动`systemctl enable docker`
