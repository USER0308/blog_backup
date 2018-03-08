# 使用 nodejs 连接 MySQL 数据库

在 ubuntu16.04 中, MySQL 服务端和客户端通过创建套接字来通信, 套接字位于 `/var/run/mysqld/mysqld.sock`.

在使用 Nodejs 时, 使用了基于 ORM 的 Sequelize.js 这个库, 但这个库默认是使用 ip 加端口来连接数据库,

```
import Sequelize from 'sequelize';
let sequelize = new Sequelize('database', 'username', 'password', {
 host: 'localhost',
 port: 3306,
 dialect: 'mysql',
 pool: {
  max: 5,
  min: 0,
  idle: 10000
 }
});
```
运行, 报错. `SequelizeConnectionRefusedError: connect ECONNREFUSED 127.0.0.1:3306`

找了很久没有找到通过套接字来连接的, 于是找官方文档看看看, 通过 dialectOptions 设置 MySQL 属性
如下:
```
import Sequelize from 'sequelize';
let sequelize = new Sequelize('database', 'username', 'password', {
 host: 'localhost',
 port: 3306,
 dialect: 'mysql',
 dialectOptions: {
  socketPath: '/var/run/mysqld/mysqld.sock' // 指定套接字文件路径
 }
 pool: {
  max: 5,
  min: 0,
  idle: 10000
 }
});
```
成功!
