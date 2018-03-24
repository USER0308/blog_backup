Script is a language

Thus 0x81 represents -1. 0x80 is another representation of zero (so called negative 0). Positive 0 is represented by a null-length vector. Byte vectors are interpreted as Booleans where False is represented by any representation of zero and True is represented by any representation of non-zero.
Leading zeros in an integer and negative zero are allowed in blocks but get rejected by the stricter requirements which standard full nodes put on transactions before retransmitting them. Byte vectors on the stack are not allowed to be more than 520 bytes long. Opcodes which take integers and bools off the stack require that they be no more than 4 bytes long, but addition and subtraction can overflow and result in a 5 byte integer being put on the stack.


多方签名会生成两个地址: P2SH 地址和 REDEEM SCRIPT, 前者用于收款, 后者用于提款
一个有效的 REDEEM SCRIPT 地址, 格式为:<OP_2><A pubkey>< B pubkey><C pubkey><OP_3><OP_CHECKMULTISIG>
前一个 OP_2 代表需要两个私钥才能使用该多方签名地址的资金, OP_3 代表创建这个多方签名地址一共需要有三个公钥
参考 https://en.bitcoin.it/wiki/Script
<OP_1> 的十进制是 81, 十六进制是 51
<OP_2> 的十进制是 82, 十六进制是 52
<OP_3> 的十进制是 83, 十六进制是 53
参考多方签名源码
```
/* new multisig address, provide the pubkeys AND required signatures to release the funds */
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

`81 + (required*1) - 1` 这句中, 如果 required = 1, 那么就是运算结果是 81, 也就是 < OP_1>, 表示需要一个人签名就可以用多方签名钱包里的钱, 如果 required = 2, 运算结果是 82, 也就是 < OP_2>, 同理. 由于 Script 语言中只定义了 < OP_1 > 到 < OP_16>, 所以最多只能 15 个人创建一个钱包
```
for (var i = 0; i < pubkeys.length; ++i) {
  s.writeBytes(Crypto.util.hexToBytes(pubkeys[i]));
}
```
这段依次把创建者的公钥写入
`s.writeOp(81 + pubkeys.length - 1); //OP_1` 这段和上上面一样, 只是这次是 length, 也就是所有参与方的人数
`s.writeOp(174); //OP_CHECKMULTISIG` 通过查询上面的百科可知,<OP_CHECKMULTISIG > 的十进制是 174, 十六进制是 ae
至此, REDEEM SCRIPT 地址创建完毕, 然后就是用这个地址生成钱包地址
先是将 REDEEM SCRIPT 地址进行两次 SHA256 hash, 提取头部四位作为摘要,
然后 REDEEM SCRIPT 地址进行 ripemd160(SHA256()) 哈希, 再在头部加上代表多方签名的标识符 "05", 再连接上摘要, 进行 base58encode
疑问: 要不要连接上摘要? coinbin 中要, 但文章中 (http://www.soroushjp.com/2014/12/20/bitcoin-multisig-the-hard-way-understanding-raw-multisignature-bitcoin-transactions/,https://zhuanlan.zhihu.com/p/30949559) 不要

-------
从个人钱包转账到多方签名地址钱包
扩展一下:
transaction

![transaction](http://ovt2bylq8.bkt.clouddn.com/d6b69478a746ccbbebc444d8793403a0.png)


![sign_transaction](http://ovt2bylq8.bkt.clouddn.com/68895623d077ae35023573c32b3a584f.png)

需要参数:
input_transaction_id,privateKey,destination,amount
输出 raw transaction

raw transaction 组成:
Version byte
input count
Previous tx hash(reversed)
Output index
scriptSign length of 138 bytes
scriptSign
  Push 71 bytes to stack
  <signature>
  Push 65 bytes to stack
  <pubKey>
Sequence
No. of outputs
Amount of 65600 in LittleEndian
scriptPubKey length of 23 bytes
scriptPubKey
  OP_HASH160
  Push 20 bytes to stack
  redeemScriptHash
  OP_EQUAL
locktime

首先去挖矿, 得到第一笔 utxo, 交易输入为这笔 utxo 的 id, 输入地址为对方钱包地址, 还包括转账金额 (和返回金额?), 然后对这笔交易签名, 广播到网络中.

A 挖矿, 得到系统给的一张支票, A 在支票上写上自己的名字, 这张支票就属于 A 的了, A 想要花费这笔钱的时候, 拿出这张支票的 ID, 写上收款人的地址, 签上自己的私钥, 告诉所有人, 大家都把这张支票复制一份, 别人想动这笔钱是不行的, 收款人想要使用这笔钱的时候, 拿出这张支票的 ID, 写上收款人的地址, 签上自己的私钥, 告诉所有人,
```
----------
支票
----------
1 块付给 X
X 的地址
----------
2 块付给 Y
Y 的地址
----------
...
----------
```
部分给, 并非全部给

> 三、交易过程
> 下面，我把整个流程串起来，看看比特币如何完成一笔交易。
> 一笔交易就是一个地址的比特币，转移到另一个地址。由于比特币的交易记录全部都是公开的，哪个地址拥有多少比特币，都是可以查到的。因此，支付方是否拥有足够的比特币，完成这笔交易，这是可以轻易验证的。
> 问题出在怎么防止其他人，冒用你的名义申报交易。举例来说，有人申报了一笔交易：地址 A 向地址 B 支付 10 个比特币。我怎么知道这个申报是真的，申报人就是地址 A 的主人？
> 比特币协议规定，申报交易的时候，除了交易金额，转出比特币的一方还必须提供以下数据。
> 上一笔交易的 Hash（你从哪里得到这些比特币）
> 本次交易双方的地址
> 支付方的公钥
> 支付方的私钥生成的数字签名
> 验证这笔交易是否属实，需要三步。
> 第一步，找到上一笔交易，确认支付方的比特币来源。
> 第二步，算出支付方公钥的指纹，确认与支付方的地址一致，从而保证公钥属实。
> 第三步，使用公钥去解开数字签名，保证私钥属实。
> 经过上面三步，就可以认定这笔交易是真实的。
>http://www.ruanyifeng.com/blog/2018/01/bitcoin-tutorial.html


>https://arstechnica.com/tech-policy/2017/12/how-bitcoin-works/
