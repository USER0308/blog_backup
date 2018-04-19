# 服务器的一些常用配置指令

每次入手服务器都要新建用户, 修改密码, 修改 root 密码, ssh 登录等等一堆琐事, 现在就记录下来, 方便下次查询.

## 创建用户

`useradd $USERNAME` 创建用户

`passwd $USERNAME` 为该用户设置密码

## 设置 root 密码 (仅首次)
新建服务器的时候, root 是禁用的, 需要设置 root 密码才能使用 root 权限

`sudo passwd root`

## 允许 root 登录

很多文章都说禁用 root 登录, 若服务器只是自己用的话, 禁不禁无所谓, 事实上, 如果服务器数量多到一定程度的话, 很难想起某个 ip 对于哪台服务器, 该服务器用户名又是什么, 对应的密码又是什么. 这就造成登录的不方便, 也许使用 ssh 密钥登录是个好方法, 万一想在新电脑远程连接服务器呢? 所以还是允许 root 登录.

`vi /etc/ssh/sshd_conf`

把 `PermitRootLogin` 设置为 `yes`

## ssh 密钥登录

上面说到了, 密码管理很麻烦, 可以使用 ssh 密钥实现免密码登录.

`ssh key-gen` 在本地生成公私钥对

`scp ~/.ssh/id_rsa.pub $USERNAME@$ip:/home/$REMOTE_USERNAME/.ssh/authorized_keys`

可能提示permission deny,可以通过vim在本地复制,然后粘贴到服务器上.
