感觉中拖了好久没写过博客了，一直都想写点东西的，但一直在找各种借口，今天一回头看上一篇博客已经是8月2日的时候发表的了。

不知不觉距离上次写博客已有一个月多了。
想想这个月我都做了些啥呢？

7月8月都在搞区块链的项目，2日之后是在做一个动态添加节点(peer或者org)进已部署好的网络中，那时候给的期限是两周，两周过后的分享会上必须拿出成果来。接这个任务的时候以为不难，因为pc刚好搞完Fabric网络，我也研究完了搭建网络的一个基本流程，要用到的配置文件等。接完任务后两天就把静态多节点的Fabric网络给部署起来了，当时还挺开心的。然后就着手搭建动态网络。

搭建多节点静态网络的时候，是使用configtxgen工具生成必要的证书密钥等文件，轮到动态添加节点的时候，想着只要像静态那样用configtxgen工具再生成一次就行了，但是事情并没有那么简单。多方尝试之下还是失败告终，无论是看出错信息或者看logs日志都无法解决这个问题。没办法，得找人问问了。去群里问，嗯，大家都有这个疑问;发邮件给专业人士，没收到回信;又回头啃全英文档，第一次把除了英语书外几厘米厚的英语“书”看完了，文档里面确实有提到动态添加节点，并且给出了一个工具configtxlator，用来将二进制文件与json文件相互转化，就是说要拿到正常运行的Fabric网络的相关二进制文件，用configtxlator转化为json文件，然后自己在json文件中修改，增加节点，保存，再使用configtxlator转化为二进制文件，替换掉原来的二进制，这样就能动态添加节点了。

官方文档中虽然说了这些，但却没有实际给出例子，也没有指出在json的哪部分修改，对应修改的格式，只说了参考文档另一个例子(该例子中，使用configtxlator修改了一个变量的数值size)来修改，对真正的动态添加节点参考意义不大，但起码也是一个方向。然后就去找configtxlator的相关资料，可谓少之又少，在github上找到了configtxlator的源码，附带使用说明--正是Fabric文档中的那部分。然后进度停在这里两天，pc说上rocketchat上找找，登录了rocketchat，找了下，找到一个引用自stackoverflow的提问，然后进stackoverflow，指出了动态添加节点的流程，流程还是那个流程，然后附加一条rocketchat的链接。当时心态真的爆炸了。就像在兜兜转转最后回到原点，说好的柳暗花明又一村的呢？

然后就是分享会了，分享会上说了自己走过的坑，最后表明暂时还没有办法实现。自然是被自己的良心折磨了好久。这个时候已经是两周过去了，进度缓慢，还有半个月就要开学了，然后老师提出再最后的两周做出个区块链demo出来，其实之前开小会的时候就已经明确了这个目标了。回来当晚加第二天一早开了个项目人员小会，明确了需求，概要设计，然后就开始实干了。用的是composer来部署网络，两个医院(Org/peer)的Fabric网络，轻车熟路就把网络配置文件写好，拉起网络，一切正常，再给my跑在几台物理机上，没事。然后Composer这边要求网络中要有CA节点，配置文件一改，拉起网络，正常。但是composer却连接不上我们这边的网络。要求提供证书密钥文件给composer，能用Node SDK连上了却无法执行进一步操作，composer仍然无法连接，怀疑是TLS的问题，关了TLS，继续重新，直接连不上。然后各路求救，内部微信群，rocketchat群问私问都找不到解决办法，根据某公司提供的信息，他们之前也使用过composer，但是由于composer还不是很完善，bug太多，他们弃用了composer，改回Node SDK。听到这个消息时，大家都很低迷，毕竟用composer快一个月了，而且也快到这个月月尾了，就这样放弃是不是太可惜了呢。

后来，zf无意中发现了一个Node SDK的sample和我们的需求很接近，阅读了一下源码，发现我们之前使用composer的时候使用了错误的证书文件，有证书ABCD，需要的是C，给的是A，然后重搭网络，给对证书，Node SDK连上了网络，能够执行基本的查询了。这件事让大家坚定了Node SDK，决定了启用composer，这样一来就意味着推到很多已有的重做。此时已是月尾，与公司那边进行一次通话会议后，他们决定派出一名本地的人员过来协助我们，然后大家互相了解一下情况，电话会议结束。然后就是把椅子风扇搬回图书馆，撤离实验室，腾出来它用。

回顾一下，后面的这半个月我负责的是前端和底层网络搭建的工作。前端是Vue，后端Koa，都是用的javascript，javascript大一的时候看过，没学会，现在又看，还是没学会，虽说在bbs项目中有用到，但感觉写出来的更像是Java而不是javascript，他们笑道我这是Java版的javascript。

再说说杂七杂八的事，8月期间用两天半完成操作系统的两个大作业，用了单例模式，用起来得心应手，用了正则表达式，感觉每次要用正则的时候都要回去温习一次/W，/d。期间断断续续完成了基于python/Django的数据库实训--bbs论坛，前端bootstrap，本来是没有的，直接html/css，后来实在是丑到我了，给加上了，bootstrap给我的感觉就是组件有点少，对了，Vue前端用了Element.ui，组件库挺丰富且优美的，就是MVVM模式不太会用，数据绑定经常出错。

其他的话，出去采访过一次，有幸见识到商业中心的繁华和写字楼的高大上，想立刻上班，但只怕那时反过来怀念校园的生活。

没了。接下来会整理一下之前的一些记录啊。
