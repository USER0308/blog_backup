## BCOS的web3sdk连接区块链测试

> 环境:Ubuntu16.04, jdk8, web3sdk V1.1.0, 新版本的eclipse(下载地址:http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/photon/RC3/eclipse-dsl-photon-RC3-linux-gtk-x86_64.tar.gz)

### 安装gradle插件

安装完打开eclipse后,安装gradle插件
在Eclipse中，点击菜单 Help -> Install New Software，将http://dist.springsource.com/release/TOOLS/gradle 粘贴到 "Work with"文本框
选择下拉框中出现的gradle - http://dist.springsource.com/release/TOOLS/gradle
选择列出的"Extensions / Gradle Integration'节点
一路点击Next直到Finish
重启eclipse

### 导入web3sdk

选择file-import-Gradle-Existing Gradle Project,定位到web3sdk目录,一路next直到finish.

然后eclipse就会查看gradle user home direction这些设置来查找gradle,如果没有的话就从网上下载gradle.之前做安卓程序的时候下载过gradle3.2版本的,但是不知道为什么现在又要下载gradle-4.3-bin.zip,从国外网站下载的,速度奇慢,挂了VPN下几次才成功,后面就下一些maven的jar依赖包.然后就可以打开web3sdk不报错.

### bcos和虚拟机配置

打开虚拟机,网络模式为桥接,上一次搭建好bcos区块链的时候是在实验室,当时初始化创世节点和普通节点的时候使用的ip地址都是当时实验室wifi分配的地址,如果现在是在别的地方做,很可能虚拟机的ip地址不同了,所以使用命令`ip addr`查看当前虚拟机ip地址,如果和之前的不一样,可能要重新搭建bcos.(有同学不重新搭建也可以,这里有疑点)搭建好bcos后确保达成共识.然后把/bcos-data/node0/data/ca.crt通过scp或者其他方式复制到web3sdk的web3sdk-1.1.0/src/test/resources/文件夹下,同时还要将web3sdk-1.1.0/client.store复制到web3sdk-1.1.0/src/test/resources/文件夹下.

### web3sdk配置文件修改

编辑applicationContext.xml文件,修改默认的ip地址和端口号为虚拟机的ip和搭建bcos时在/bcos-data/node0/config.json配置文件中指定的channelport的端口52300.
在虚拟机中使用`firewall-cmd --state`查看防火墙状态,发现正在运行,说明开启了防火墙,使用`firewall-cmd --list-ports`查看开放的端口,发现只有`53300/tcp,52300/tcp和35500/tcp`,而我之前在applicationContext.xml中指定的channelPort为节点node1的52301,所以没有开放出来.这可以在宿主机使用`telnet 虚拟机ip 52301`发现端口能否被访问.正是因为端口没开放,访问不了,才导致报以下错误
```
java.lang.Exception
	at org.bcos.channel.client.Service.run(Service.java:242) [bin/:?]
	at org.bcos.channel.test.Ethereum.main(Ethereum.java:26) [bin/:?]
2018-06-22 20:25:23.676 [main] ERROR Service() - connectSeconds = 10
2018-06-22 20:25:23.676 [main] ERROR Service() - init ChannelService fail!
2018-06-22 20:25:23.676 [main] ERROR Service() - system error 
```
### 其他报错

运行web3sdk-1.1.0/src/test/java/org.bcos.channel.test/Ethereum.java的时候还有可能会报`cannot write to /data/app/logs/bsp-ebcfs/*.log 权限不够`,因为/data文件夹是root用户所有,使用普通用户去执行java文件当然权限不够,解决办法是将bsp-ebcfs文件夹拥有者从root改为当前用户.命令为`chown -R user0308:user0308 /data/app/logs/bsp-ebcfs/`,当然了,因为这个路径是在web3sdk-1.1.0/src/test/resources/log4j2.xml中定义的,也可以修改log4j2.xml中的路径为当前用户家目录下的某个目录.
由于指定了log日志输出的目录,所以日志会输出在log文件中而非控制台,找错需要看error.log.

### 成功运行输出

最后,运行test中的Ethereum.java,输出

```
Jun 22, 2018 9:16:34 PM org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
INFO: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@74650e52: startup date [Fri Jun 22 21:16:34 CST 2018]; root of context hierarchy
Jun 22, 2018 9:16:34 PM org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
INFO: Loading XML bean definitions from class path resource [applicationContext.xml]
Jun 22, 2018 9:16:35 PM org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor initialize
INFO: Initializing ExecutorService  'pool'
开始测试...
===================================================================
Ok getContractAddress 0xe5b20984d46bc4457b72f1511e343ae207e9e0d6
receipt transactionHash0xf7f8e978359aef5c677fe85b6ff29a73590d914f87b2f0fe6f666422bed2e8c2
ok.get() 999
```
### 附录:

建议使用eclipse而不是IDEA来运行,首先build.gradle就明确指明了apply plugin: 'eclipse',用IDEA的话会出现各种classpath错误,找不到路径等等

