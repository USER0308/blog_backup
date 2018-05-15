# 比特币技术实现

## 钱包地址

钱包通过一串字符区分,钱包有两种,一种是个人钱包,属于个人所有,个人可以自由支配钱包中的余额;另一种是多方钱包,属于几个或十几个人联合所有,可以随时接受收入,但支出必须经过这几个或十几个人中部分或全部同意方可.钱包字符串长度为58,以'1'开头的是个人钱包地址,以'3'开头的是多方钱包地址.58个地址字符是通过Base58编码算法将长度压缩后得到,目的是减少传输过程中因字符串长度太长导致看错/记错/写错等.58个字符串由"123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz"构成,除去了0,O,I,l这几个可能混淆的字符.钱包地址是由公钥经过Base58编码转化而来.比特币系统中,每个用户都有一对密钥,一个是公钥,一个是私钥,公钥相当于一把锁,私钥就是锁对应的钥匙.私钥由比特币系统采集一系列随机数再经过多次调用sha256产生,所以私钥的长度为256 bits,即64位十六进制字符串.有了私钥,使用椭圆曲线加密(ECC)算法生成私钥唯一对应的公钥.由于私钥的随机性,单靠记忆很难将之记住,所以需要将私钥以文件的形式保存到磁盘中.

### ECC 公私钥

椭圆曲线加密算法, 具体的介绍见 [这里](http://pangjiuzala.github.io/2016/03/03/Bitcoin%E5%8A%A0%E5%AF%86%E6%8A%80%E6%9C%AF%E4%B9%8B%E6%A4%AD%E5%9C%86%E6%9B%B2%E7%BA%BF%E5%AF%86%E7%A0%81%E5%AD%A6/)
另外一篇简单介绍椭圆曲线加密算法:
>
> 作者：知乎用户
  > 链接：https://www.zhihu.com/question/26662683/answer/325511510
> 来源：知乎
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
>
> 椭圆曲线加密
>
>
> 考虑 K=kG ，其中 K、G 为椭圆曲线 Ep(a,b) 上的点，n 为 G 的阶（nG=O∞），k 为小于 n 的整数。则给定 k 和 G，根据加法法则，计算 K 很容易, 但反过来，给定 K 和 G，求 k 就非常困难。因为实际使用中的 ECC 原则上把 p 取得相当大，n 也相当大，要把 n 个解点逐一算出来列成上表是不可能的。这就是椭圆曲线加密算法的数学依据. 点 G 称为基点（base point）k（k<n）为私有密钥（privte key）K 为公开密钥（public key)

>
> ECC 保密通信算法
>
> * 1.Alice 选定一条椭圆曲线 E，并取椭圆曲线上一点作为基点 G  假设选定 E29(4,20)，基点 G(13,23) , 基点 G 的阶数 n=37
> * 2.Alice 选择一个私有密钥 k（k<n），并生成公开密钥 K=kG   比如 25, K= kG = 25G = (14,6）
> * 3.Alice 将 E 和点 K、G 传给 Bob
> * 4.Bob 收到信息后，将待传输的明文编码到上的一点 M（编码方法略），并产生一个随机整数 r（r<n,n 为 G 的阶数）   假设 r=6  要加密的信息为 3, 因为 M 也要在 E29(4,20) 所以 M=(3,28)
> * 5.Bob 计算点 C1=M+rK 和 C2=rG  C1= M+6K= M+6*25*G=M+2G=(3,28)+(27,27)=(6,12)  C2=6G=(5,7)
> * 6.Bob 将 C1、C2 传给 Alice7.Alice 收到信息后，计算 C1-kC2，结果就应该是点 M  C1-kC2 =(6,12)-25C2 =(6,12)-25*6G =(6,12)-2G =(6,12)-(27,27) =(6,12)+(27,2) =(3,28) 数学原来上能解密是因为: C1-kC2=M+rK-krG=M+rkG-krG-M
>
> 比特币系统选用的 secp256k1 中，参数为
> p = 0xFFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFE FFFFFC2F = 2^256 − 2^32 − 2^9 − 2^8 − 2^7 − 2^6 − 2^4 − 1
> a = 0，
> b = 7
> G=(0x79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798, 0x483ada7726a3c4655da4fbfc0e1108a8fd17b448a68554199c47d08ffb10d4b8)
> n = 0xFFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFE BAAEDCE6 AF48A03B BFD25E8C D0364141
> h = 01

![2018-04-23-21-09-20](2018-04-23-21-09-20.png)

## 挖矿

### PoW 共识算法

### P2P 网络

### 分叉

## 交易

### 创建空白交易

### 对交易签名

### Script

### 多方签名

Appendex:
https://en.bitcoin.it/wiki/Technical_background_of_version_1_Bitcoin_addresses
https://en.bitcoin.it/wiki/Wallet_import_format
https://www.jianshu.com/p/af6328cc693e
https://www.jianshu.com/p/07693c746aa9
https://www.jianshu.com/p/c792aa0e273d
https://www.jianshu.com/p/a138b20dd1ae

每次想写点东西,结果网上都能找到更好的,这极大地打击了原创的心 :(