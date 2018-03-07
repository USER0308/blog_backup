# 多方签名算法研究

    本课题研究目前的多方签名算法, 并用 C++ 实现.

多方签名, 可以理解为一个数字资产的多个签名, 比如转账时需要获得多个参与方的授权来执行交易. 如果要动用一个加密货币地址的资金, 通常需要该地址的所有人使用他的私钥 (有用户专属保护) 进行签名. 那么, 多重签名, 就是动用这笔资金或数字资产会保存在一个多重签名的地址或帐号里. 这就好比, 我们工作中有一份文件, 需要多个部门签署才能生效一样. 在实际的操作过程中, 一个多重签名地址可以关联 n 个私钥, 在需要转账等操作时, 只要其中的 m 个私钥签名就可以把资金转移了, 其中 m 要小于等于 n, 也就是说 m/n 小于 1, 可以是 2/3,3/5 等等, 是要在建立这个多重签名地址的时候确定好的.
建议能用 C++ 结合现有的签名算法实现一个区块链转账场景下的多重签名实例.

首先获得的提示只有上面的第一段的那一句话, 感觉太过笼统, 无从下手, 于是又和需求方询问, 得到的回答是第二段的长篇幅回答, 第二个回答具体回答了什么是多方签名, 这个回答还是有些模糊, 关键句子在于最后一句, 建议能用 C++ 结合现有的签名算法实现一个区块链转帐场景下的多重签名实例. 抛开编程语言 C++ 不说, 关键点有: 现有的签名算法, 区块链转账场景实例. 签名算法是指现有的多方签名算法还是现有的单方签名算法? 区块链转账场景具体是指什么? 是指自己搭建一个私有区块链网络, 然后在上面实现多方签名实例? 还是指做一个模拟区块链运行的 demo, 省去大量细节, 只包含创建帐号和转账等基本功能, 然后实现多方签名吗? 如果是指前者, 那么规定编程语言为 C++ 意义在何处? 因为前者完全可以通过引入一个已有的成熟的多方签名智能合约实现其功能, 而智能合约是用 solidity 等非 C++ 语言编写. 如果是后者, 使用 C++ 模拟区块链运行就说得通了, 同时无法引入已有的多方签名智能合约, 只能自己实现多方签名底层细节打包成函数供上层调用.

现有的签名算法

普通找到的

比特币使用的
http://www.wanbizu.com/baike/201408191710.html
http://www.mingin.com/btc/news/10225-1.html
具体非官方代码:
https://github.com/OutCast3k/coinbin/tree/master/js
参与者提供各自的 34 位钱包地址和 66 位公钥

多方签名地址以 3 开头, 个人钱包地址以 1 开头
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

demo

http://www.soroushjp.com/2014/12/20/bitcoin-multisig-the-hard-way-understanding-raw-multisignature-bitcoin-transactions/

https://github.com/soroushjp/go-bitcoin-multisig


appendix
https://bitcoinmagazine.com/articles/multisig-future-bitcoin-1394686504/
以太坊使用的
以太坊可以使用 Mist 创建多方签名钱包地址, 实质是使用一个个人钱包地址调用一个智能合约, 把剩下参与方的公钥都作为参数传给智能合约.
http://book.8btc.com/books/6/ethereum/_book/create-security-signature-wallet.html
该智能合约为:
https://raw.githubusercontent.com/ethereum/dapp-bin/master/wallet/wallet.sol
在这篇文章中提到上述智能合约 outdated,updated is
https://ethereum.stackexchange.com/questions/6827/wheres-the-solidity-code-for-mists-default-multi-sig-contract-wallet
