# 多方签名算法研究

    本课题研究目前的多方签名算法, 并用 C++ 实现.

多方签名, 可以理解为一个数字资产的多个签名, 比如转账时需要获得多个参与方的授权来执行交易. 如果要动用一个加密货币地址的资金, 通常需要该地址的所有人使用他的私钥 (有用户专属保护) 进行签名. 那么, 多重签名, 就是动用这笔资金或数字资产会保存在一个多重签名的地址或帐号里. 这就好比, 我们工作中有一份文件, 需要多个部门签署才能生效一样. 在实际的操作过程中, 一个多重签名地址可以关联 n 个私钥, 在需要转账等操作时, 只要其中的 m 个私钥签名就可以把资金转移了, 其中 m 要小于等于 n, 也就是说 m/n 小于 1, 可以是 2/3,3/5 等等, 是要在建立这个多重签名地址的时候确定好的.
建议能用 C++ 结合现有的签名算法实现一个区块链转账场景下的多重签名实例.

首先获得的提示只有上面的第一段的那一句话, 感觉太过笼统, 无从下手, 于是又和需求方询问, 得到的回答是第二段的长篇幅回答, 第二个回答具体回答了什么是多方签名, 这个回答还是有些模糊, 关键句子在于最后一句, 建议能用 C++ 结合现有的签名算法实现一个区块链转帐场景下的多重签名实例. 抛开编程语言 C++ 不说, 关键点有: 现有的签名算法, 区块链转账场景实例. 签名算法是指现有的多方签名算法还是现有的单方签名算法? 区块链转账场景具体是指什么? 是指自己搭建一个私有区块链网络, 然后在上面实现多方签名实例? 还是指做一个模拟区块链运行的 demo, 省去大量细节, 只包含创建帐号和转账等基本功能, 然后实现多方签名吗? 如果是指前者, 那么规定编程语言为 C++ 意义在何处? 因为前者完全可以通过引入一个已有的成熟的多方签名智能合约实现其功能, 而智能合约是用 solidity 等非 C++ 语言编写. 如果是后者, 使用 C++ 模拟区块链运行就说得通了, 同时无法引入已有的多方签名智能合约, 只能自己实现多方签名底层细节打包成函数供上层调用.

最终决定:
模拟场景精化描述:
  使用 C++ 编写多方签名算法实现转账场景, 场景包括: 提供密钥来生成公钥和个人钱包地址, 使用多方 (n 方) 的公钥生成共有钱包地址, 从个人钱包转账到共有钱包, 通过 m(m<=n) 个密钥解锁共有钱包转账给私有钱包.
对主流算法研究:
  对现有的三种多方签名算法研究分析其优缺点.


现有的签名算法

普通找到的

比特币使用的
http://www.wanbizu.com/baike/201408191710.html
http://www.mingin.com/btc/news/10225-1.html
具体非官方代码:
https://github.com/OutCast3k/coinbin/tree/master/js


多方签名地址以 3 开头, 个人钱包地址以 1 开头
以比特币系统为例, 其非对称加密机制如图 4 所示: 比特币系统一般通过调用操作系统底层的随机数生成器来生成 256 位随机数作为私钥. 比特币私钥的总量可达 22562256, 极难通过遍历全部私钥空间来获得存有比特币的私钥, 因而是密码学安全的. 为便于识别, 256 位二进制形式的比特币私钥将通过 SHA256 哈希算法和 Base58 转换, 形成 50 个字符长度的易识别和书写的私钥提供给用户; 比特币的公钥是由私钥首先经过 Secp256k1 椭圆曲线算法生成 65 字节长度的随机数. 该公钥可用于产生比特币交易时使用的地址, 其生成过程为首先将公钥进行 SHA256 和 RIPEMD160 双哈希运算并生成 20 字节长度的摘要结果 (即 hash160 结果), 再经过 SHA256 哈希算法和 Base58 转换形成 33 字符长度的比特币地址 [19]. 公钥生成过程是不可逆的, 即不能通过公钥反推出私钥.
wiki 上的流程:
0 - Having a private ECDSA key   64bytes

  18E14A7B6A307F426A94F8114701E7C8E774E7F9A47E2C2035DB29A206321725
1 - Take the corresponding public key generated with it (65 bytes, 1 byte 0x04, 32 bytes corresponding to X coordinate, 32 bytes corresponding to Y coordinate)

   0450863AD64A87AE8A2FE83C1AF1A8403CB53F53E486D8511DAD8A04887E5B23522CD470243453A299FA9E77237716103ABC11A1DF38855ED6F2EE187E9C582BA6
2 - Perform SHA-256 hashing on the public key

   600FFE422B4E00731A59557A5CCA46CC183944191006324A447BDB2D98D4B408
3 - Perform RIPEMD-160 hashing on the result of SHA-256

   010966776006953D5567439E5E39F86A0D273BEE
4 - Add version byte in front of RIPEMD-160 hash (0x00 for Main Network)

   00010966776006953D5567439E5E39F86A0D273BEE
(note that below steps are the Base58Check encoding, which has multiple library options available implementing it)
5 - Perform SHA-256 hash on the extended RIPEMD-160 result

   445C7A8007A93D8733188288BB320A8FE2DEBD2AE1B47F0F50BC10BAE845C094
6 - Perform SHA-256 hash on the result of the previous SHA-256 hash

   D61967F63C7DD183914A4AE452C9F6AD5D462CE3D277798075B107615C1A8A30
7 - Take the first 4 bytes of the second SHA-256 hash. This is the address checksum

   D61967F6
8 - Add the 4 checksum bytes from stage 7 at the end of extended RIPEMD-160 hash from stage 4. This is the 25-byte binary Bitcoin Address.

   00010966776006953D5567439E5E39F86A0D273BEED61967F6
9 - Convert the result from a byte string into a base58 string using Base58Check encoding. This is the most commonly used Bitcoin Address format

   16UwLL9Risc3QfPqBUvKofHmBQ7wMtjvM

代码实现:
```
coinjs.newPubkey = function(hash){
		var privateKeyBigInt = BigInteger.fromByteArrayUnsigned(Crypto.util.hexToBytes(hash));
		var curve = EllipticCurve.getSECCurveByName("secp256k1");

		var curvePt = curve.getG().multiply(privateKeyBigInt);
		var x = curvePt.getX().toBigInteger();
		var y = curvePt.getY().toBigInteger();

		var publicKeyBytes = EllipticCurve.integerToBytes(x, 32);
		publicKeyBytes = publicKeyBytes.concat(EllipticCurve.integerToBytes(y,32));
		publicKeyBytes.unshift(0x04);

		if(coinjs.compressed==true){
			var publicKeyBytesCompressed = EllipticCurve.integerToBytes(x,32)
			if (y.isEven()){
				publicKeyBytesCompressed.unshift(0x02)
			} else {
				publicKeyBytesCompressed.unshift(0x03)
			}
			return Crypto.util.bytesToHex(publicKeyBytesCompressed);
		} else {
			return Crypto.util.bytesToHex(publicKeyBytes);
		}
	}

	/* provide a public key and return address */
	coinjs.pubkey2address = function(h, byte){
		var r = ripemd160(Crypto.SHA256(Crypto.util.hexToBytes(h), {asBytes: true}));
		r.unshift(byte || coinjs.pub);
		var hash = Crypto.SHA256(Crypto.SHA256(r, {asBytes: true}), {asBytes: true});
		var checksum = hash.slice(0, 4);
		return coinjs.base58encode(r.concat(checksum));
	}
```
多方签名代码:
```
coinjs.pubkeys2MultisigAddress = function(pubkeys, required) {
  var s = coinjs.script();
  s.writeOp(81 + (required*1) - 1); //OP_1
  for (var i = 0; i < pubkeys.length; ++i) {
    s.writeBytes(Crypto.util.hexToBytes(pubkeys[i]));
  }
  s.writeOp(81 + pubkeys.length - 1); //OP_1
  s.writeOp(174); //OP_CHECKMULTISIG
  var x = ripemd160(Crypto.SHA256(s.buffer, {asBytes: true}), {asBytes: true});
  x.unshift(coinjs.multisig);
  var r = x;
  r = Crypto.SHA256(Crypto.SHA256(r, {asBytes: true}), {asBytes: true});
  var checksum = r.slice(0,4);
  var redeemScript = Crypto.util.bytesToHex(s.buffer);
  var address = coinjs.base58encode(x.concat(checksum));

  if(s.buffer.length> 520){ // too large
    address = 'invalid';
    redeemScript = 'invalid';
  }

  return {'address':address, 'redeemScript':redeemScript, 'size': s.buffer.length};
}
```
![btc](http://ovt2bylq8.bkt.clouddn.com/44319eb08d66ce4f24f3b93e91372874.png)
附加比特币多方签名 demo(仅调用比特币函数)
https://github.com/johnsondiao/blackboard101

./bitcoind createmultisig 2 ["","",""]
./bitcoind createrawtransaction [{}]
./bitcoind signrawtransaction $addr
./bitcoind decoderawtransaction $addr
repead create sign and decode

详细介绍

http://www.soroushjp.com/2014/12/20/bitcoin-multisig-the-hard-way-understanding-raw-multisignature-bitcoin-transactions/

代码例子:
https://github.com/soroushjp/go-bitcoin-multisig

过程:
### 生成多方签名地址
提供 n 个 16 进制格式的公钥, 现有许多生成公钥 / 密钥对的工具, 但是在这里我们使用了集成在 go-bitcoin-multisig 中的一个. 这样的公钥 / 密钥对在密码学上足够安全, 因为使用了 go 语言的 crypto/rand 包, 一个使用了在类 unix 系统中的 / dev/urandom 的 API 或在 Windows 系统的 CryptoGenRandom 的 API.

根据比特币协议 [!protocol](https://bitcoin.org/en/developer-guide#standard-transactions), 一个有效的 multisignature redeem script 看起来像这样:<OP_2><A pubkey><B pubkey><C pubkey><OP_3><OP_CHECKMULTISIG>

P2SH address 通过另外两步来生成.
第一步: 对 redeem script 进行两次哈希算法, 第一次是将 redeem script 进行 SHA256 哈希, 第二次是将第一次得到的结果再进行 RIPEMD160 哈希
第二步: 对上面得到的结果加上前缀 (0X05), 转换成 Base58Check 编码

### 转账到多方签名地址
想要转账到多方签名地址, 我们需要一个比特币转账源. go-bitcoin-multisig 可以从标准的 P2PKH 转账, 我们需要交易 Id(transaction id,txid), 对应的私钥, 转账额度 (多余的额度将作为小费), 目标地址 P2SH(刚刚生成的) 作为输入,
appendix
https://bitcoinmagazine.com/articles/multisig-future-bitcoin-1394686504/
以太坊使用的
以太坊可以使用 Mist 创建多方签名钱包地址, 实质是使用一个个人钱包地址调用一个智能合约, 把剩下参与方的公钥都作为参数传给智能合约.
http://book.8btc.com/books/6/ethereum/_book/create-security-signature-wallet.html
该智能合约为:
https://raw.githubusercontent.com/ethereum/dapp-bin/master/wallet/wallet.sol
在这篇文章中提到上述智能合约 outdated,updated is
https://ethereum.stackexchange.com/questions/6827/wheres-the-solidity-code-for-mists-default-multi-sig-contract-wallet


多方签名算法研究:
面向多用户的无证书数字签名方案研究
http://cdmd.cnki.com.cn/Article/CDMD-10358-1015723060.htm
一种基于 ECDSA 的有序多重数字签名方案
有序多重签名 (广播多重签名见论文附录 5)
http://www.cqvip.com/qk/87339a/201602z/68789083504849544853485656.html
电子合同签署中有序多重数字签名的应用
http://www.cqvip.com/read/read.aspx?id=668058602
更快速更高效的基于超椭圆曲线双线性对用于数字签名
(基于 RSA 的有序和广播签名方案见附录 2, 椭圆曲线双线性引用 4-6)
http://www.cqvip.com/qk/88997x/201661/669544274.html
基于 Koblitz 曲线的数字签名研究
http://cdmd.cnki.com.cn/Article/CDMD-10060-1015367328.htm



目前比特币使用的是 ECDSA 椭圆曲线数字签名算法
即将替换的是 Schnorr 签名
在这种情况下，Schnorr 签名的大小随签名者的数量呈线性增长。为了有用和实用，Schnorr 签名方案的产生与签名者的数量无关，并且只是接近于一个普通签名方案的签名而已。”
很多密码学家认为 Schnorr 签名在相关应用中是最好的，因为它有很高水平的正确性，没有延展性问题，验证速度相当快，最重要的是支持多重签名：可以把多个签名聚合成一个新的签名。
然而，到目前为止，Schnorr 签名还不能在比特币中使用，另外一种类型的签名策略——椭圆曲线数字签名算法（ECDSA）集成在比特币协议中，改变这些需要硬分叉。

隔离见证此时就发挥了作用。

隔离见证会把所有的签名数据转移到一个交易上的独立空间中：见证数据（witness）将不会被嵌入 “老的” 比特币协议中。多亏有了脚本版本控制，所有见证中的规则都可以通过软分叉来改变，包括这种签名策略的类型。

首先，签名数量减少了，每个区块中写入的交易数据容量就能增加。其次，签名融合之后，交易来源就很难被查到，因此该项技术还能提高隐私性。

缺点:
Unfortunately, unlike ECDSA, the Schnorr algorithm has not been standardized since its invention, likely because of the original patent enforced on it (which has since expired). While the general outlines of the system are mathematically sound, the lack of documentation and specification makes it more challenging to implement. Specifically, its application to the ephemeral keypairs design of Bitcoin involves security considerations that require further optimization.

有关Schnorr链接:
https://bitcoincore.org/en/2017/03/23/schnorr-signature-aggregation/

https://medium.com/@SDWouters/why-schnorr-signatures-will-help-solve-2-of-bitcoins-biggest-problems-today-9b7718e7861c

https://bitcoinmagazine.com/articles/the-power-of-schnorr-the-signature-algorithm-to-increase-bitcoin-s-scale-and-privacy-1460642496/

https://bitcoin.stackexchange.com/questions/34288/what-are-the-implications-of-schnorr-signatures/36936

https://www.reddit.com/r/Bitcoin/comments/7d5zbc/finally_real_privacy_for_bitcoin_transactions/dpvsjnm/


自己测试:
生成公私钥:
用户 1:
address:
1HDPBHffZmqEpYkj72Q73dj49yfkKCXXL8
public key:
036f1a42c28ac8372329fcdf2bda7ec6cccc3af6c7e1e7d5731eba7a4b8a6c2781
private key:
L3QMGJYk3s8bFdNMPSCNttxVazMUUW56EbaEwjA5f3vCm8yU9zPX

用户 2:
address:
1CtNHtfEWsyU5auXeLKwgBPKhq65ygDiGL
public key:
02e6271c2c67fa50a9e776285bc9b03057caa0cf018e88c757fbe2303c525d66e8
private key:
L5652sT2YDh59foP71Cp84e4nABTaQxeiNDHqttSEtJGtfQ76pyF

用户 3:
address:
1DRnZu4UoRcQa9NKh2B9knoMEQZiJgtQjv
public key:
02e43506b90a1989239f6fe08fa7585aabd6cedd5b8f9d2f237f265d1c57af0c5e
private key:
L4H87yiZiG9F9pH3ZDsPLtaY9iad22GjFBebKYQyV1fKmSZPj8Zi

share address:
3C8Q7SbAPoxZyRmy8GBSHhdrU5BVLFzne1
redeem script:
5221036f1a42c28ac8372329fcdf2bda7ec6cccc3af6c7e1e7d5731eba7a4b8a6c27812102e6271c2c67fa50a9e776285bc9b03057caa0cf018e88c757fbe2303c525d66e82102e43506b90a1989239f6fe08fa7585aabd6cedd5b8f9d2f237f265d1c57af0c5e53ae
url:
https://coinb.in/?verify=5221036f1a42c28ac8372329fcdf2bda7ec6cccc3af6c7e1e7d5731eba7a4b8a6c27812102e6271c2c67fa50a9e776285bc9b03057caa0cf018e88c757fbe2303c525d66e82102e43506b90a1989239f6fe08fa7585aabd6cedd5b8f9d2f237f265d1c57af0c5e53ae#verify
