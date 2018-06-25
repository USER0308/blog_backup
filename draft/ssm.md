## 环境

* jdk 1.8
* ubuntu 16.04
* mysql 5.7.22

## 背景知识

* ssm主要包结构:
    * entity实体层
    * dao数据访问层
    * service服务层
    * serviceImpl服务实现层

* 配置文件
    * applicationContext.xml
    * mybatis-config.xml
    * pom.xml
    * dao.xml

* schema.sql
* jdbc.properties

* jar依赖包
    * mybatis-spring-1.3.0
    * mybatis-3.4.1
    * mysql-connector-java-5.1.39
    * spring-jdbc-4.2.0-RELEASE
    * spring-core-4.2.0-RELEASE
    * JUnit4
...

## 操作

1. 在entity中编写实体类,包含属性,setter函数,getter函数

2. 在dao编写访问实体类的接口

3. 在dao.xml中编写对应的数据库操作语句

4. 在service中编写面向服务的方法,调用dao中的接口,dao.xml中具体实现

5. 在serviceImpl中具体实现服务

## 配置文件解释

### applicationContext.xml

主要是注册各种bean,识别jdbc.properties文件,根据该文件配置数据源,注入sqlSessionFactory,再根据工厂生成session,连接到对象

### mybatis-config.xml

绑定实体所在的包

### pom.xml

定义了maven,spring,mybaits的各种依赖jar

### dao.xml

与dao类强对应,实现了类中的数据库操作

### schema.sql

建数据库,建表格,初始化数据库数据

### jdbc.properties

连接数据库所需的属性,如数据库连接地址,数据库连接驱动,用户名,密码,其他配置

## 实战

建立entity包

创建Book.java类,三个属性(bookId,bookName,bookPrice),setter,getter方法

创建dao包

创建IBookDAO.java接口,明确我们要提供的服务是插入书籍记录,根据书名查询书籍记录,将这些服务细分到数据库层面,因此有两个函数(addBook,queryBook),一般来说复杂应用中一个服务可能对应多个数据库操作,比如说创建用户,需要先查询是否已存在,无则创建

创建IBookDAO.xml,将IBookDAO.java与数据库操作绑定起来

创建service包

创建IBookService.java服务接口,明确我们要提供的服务是插入书籍记录,根据书名查询书籍记录,因此有两个函数(addBook,queryBook)

创建service.impl包

创建BookServiceImpl.java类,添加@Autowired(required=false)前缀,从applicationContext.xml中获得已注入的IBookDAO对象,调用IBookDAO提供的基于数据库的接口方法(若一个服务需要用到多个DAO方法,此处需要调用多个)

创建JUnit测试BookServiceImpl.java类BookServiceImplTest.java

## 附录:

(1)
我的ubuntu使用的是`/var/run/mysqld/mysqld.sock`来连接数据库,但是在`/etc/mysql/mysql.conf.d/mysqld.cnf`中绑定了地址`127.0.0.1`,所以也可以用`127.0.0.1`来连接数据库,所以jdbc.properties中使用了`127.0.0.1`.注意:使用`localhost`可能导致无法连接数据库.

(2)
IBookDAO.xml与IBookDAO.java函数的函数名,参数必须一一对应.xml中标签与实际操作必须对应,表名,列名可用``括起来,VALUES值则用''括起来.

(3)
IBookDAO.java中接口函数为标准数据类型时需要加上`@Param("XXXX")`,XXXX与IBookDAO.xml中参数名一致
