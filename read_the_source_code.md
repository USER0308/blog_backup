# coinbin 源码解读

> coinbin 项目地址: https://github.com/coinbin/coinbin
> A Open Source Browser Based Bitcoin Wallet.
> Coinb.in supports a number of key features such as:
>
> * Offline Compressed & uncompressed Address creation.
> * Offline Multisignature Address creation.
> * Supports compressed and uncompressed keys.
> * "In browser" Key (re)generation.
> * Ability to decode transactions, redeem scripts and more offline.
> * Send and receive payments.
> * Build custom transactions offline.
> * Sign transactions offline.
> * Broadcast custom transactions.

首先看文件结构:
```
.
|--css/
|--fonts/
|--js/
|---|--aes.js
|---|--bootstrap.min.js
|---|--bootstrap-datetimepicker.js
|---|--coin.js
|---|--coinbin.js
|---|--collapse.js
|---|--crypto.min.js
|---|--crypto-sha256.js
|---|--crypto-sha256-hmac.js
|---|--ellipticcurve.js
|---|--jquery-1.9.1.min.js
|---|--jsbn.js
|---|--moment.min.js
|---|--qcode-decoder.min.js
|---|--qrcode.js
|---|--ripemd160.js
|---|--sha512.js
|---|--transition.js
|--images/
|--index.html
|--LICENSE
|--README.md
|--sha1sum
```

主要文件是 index.html 和 js 文件夹中的 coin.js,coinbin.js,crypto.\*.js,ripemd160.js
index.html 主要是使用 bootstrap 组件来编写, 然后在 coinbin.js 里绑定事件, 触发事件调用 coin.js 中的函数, coin.js 中又会调用其他 js 文件中的辅助函数 (加密解密 / 编码解码 / 进制转换等)

下面看主要的几个功能:

## 功能 1: 生成 public key/private key

首先在浏览器中打开 index.html, 选择 `New`->`address`
![new address](http://ovt2bylq8.bkt.clouddn.com/fb4cf957c4a6f394e8223393f87208fc.png)
右键点击 Generate 按钮, 审查元素, 获得按钮的 id 为 `newKeysBtn`

`<input type="button" class="btn btn-primary" value="Generate" id="newKeysBtn" _vimium-has-onclick-listener=""data-original-title="" title="">`
回到 coinbin.js,`ctrl + F` 查找 `newKeysBtn`, 找到点击该按钮触发的函数
```
$("#newKeysBtn").click(function(){
		coinjs.compressed = false;
		if($("#newCompressed").is(":checked")){
			coinjs.compressed = true;
		}
		var s = ($("#newBrainwallet").is(":checked")) ? $("#brainwallet").val() : null;
		var coin = coinjs.newKeys(s);
		$("#newBitcoinAddress").val(coin.address);
		$("#newPubKey").val(coin.pubkey);
		$("#newPrivKey").val(coin.wif);

		/* encrypted key code */
		if((!$("#encryptKey").is(":checked")) || $("#aes256pass").val()==$("#aes256pass_confirm").val()){
			$("#aes256passStatus").addClass("hidden");
			if($("#encryptKey").is(":checked")){
				$("#aes256wifkey").removeClass("hidden");
			}
		} else {
			$("#aes256passStatus").removeClass("hidden");
		}
		$("#newPrivKeyEnc").val(CryptoJS.AES.encrypt(coin.wif, $("#aes256pass").val())+'');

	});
```
在该函数中, 调用了 coinjs.newKeys(s), 参数 s 在勾选 `brainwallet` 时有效, barinwallet 就相当于我们平时的密码, 在脑中想一个密码, 系统根据这个密码生成一个帐号, 如果不提供密码, 会直接生成帐号和密码 (公钥 / 私钥), 此处参数 s 为 null.
再看 coinjs 中的 newKey 函数,
```
/* generate a private and public keypair, with address and WIF address */
	coinjs.newKeys = function(input){
		var privkey = (input) ? Crypto.SHA256(input) : this.newPrivkey();
		var pubkey = this.newPubkey(privkey);
		return {
			'privkey': privkey,
			'pubkey': pubkey,
			'address': this.pubkey2address(pubkey),
			'wif': this.privkey2wif(privkey),
			'compressed': this.compressed
		};
	}
```
在这里,`input` 值为 `null`, 所以先调用 `newPrivatekey` 函数产生一个私钥, 然后调用 `newPubkey` 函数把私钥作为参数传进去, 得到公钥, 调用 `pubkey2address`, 传入 `pubkey` 得到钱包地址, 调用 `privkey2wif`, 传入 `privkey` 得到 `WIF` 地址 (Wallet Import Format), 把 (公钥 / 私钥 / 钱包地址 / wif 地址 / 是否压缩) 作为对象属性, 返回一个对象, 然后在 coinbin.js 中将钱包地址 / 公钥 / WIF 地址填到对应的文本框中
```
$("#newBitcoinAddress").val(coin.address);
$("#newPubKey").val(coin.pubkey);
$("#newPrivKey").val(coin.wif);
```
![encrypt text](http://ovt2bylq8.bkt.clouddn.com/8e5d0014215daa0f23c023dbccd7fa27.png)
值得注意的是, 还有一个隐藏文本框, id 为 encryptKey, 只有在勾选上 `Encrypt Private Key with AES-256 Password` 的时候才会显示出来, 并且需要填写加密用的密码, 文本框的内容是使用密码加密之后的 WIF 地址.
现在来研究 newPrivKey,newPubKey,pubkey2address,privkey2wif 这几个函数
```
/* generate a new random private key */
	coinjs.newPrivkey = function(){
		var x = window.location;
		x += (window.screen.height * window.screen.width * window.screen.colorDepth);
		x += coinjs.random(64);
		x += (window.screen.availHeight * window.screen.availWidth * window.screen.pixelDepth);
		x += navigator.language;
		x += window.history.length;
		x += coinjs.random(64);
		x += navigator.userAgent;
		x += 'coinb.in';
		x += (Crypto.util.randomBytes(64)).join("");
		x += x.length;
		var dateObj = new Date();
		x += dateObj.getTimezoneOffset();
		x += coinjs.random(64);
		x += (document.getElementById("entropybucket")) ? document.getElementById("entropybucket").innerHTML : '';
		x += x+''+x;
		var r = x;
		for(i=0;i<(x).length/25;i++){
			r = Crypto.SHA256(r.concat(x));
		}
		var checkrBigInt = new BigInteger(r);
		var orderBigInt = new BigInteger("fffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141");
		while (checkrBigInt.compareTo(orderBigInt) >= 0 || checkrBigInt.equals(BigInteger.ZERO) || checkrBigInt.equals(BigInteger.ONE)) {
			r = Crypto.SHA256(r.concat(x));
			checkrBigInt = new BigInteger(r);
		}
		return r;
	}
```
有没有觉得稀里糊涂的? 感觉上是随机获取各种字符串, 再通过一定的密码学变换, 最后通过 hash256 得到结果, 看了源码也不太知道到底干了些什么, 于是打开浏览器的控制台, 运行一下
```
priv_key = coinjs.newPrivKey()
console.log(priv_key)
console.log(typeof priv_key)
```

输出
`string`
`19bcd3173cb7480356ba16d47c6d6e2ce04c9ea9d358ba39b58a4bbce8b92a44`

一共 64 位十六进制字符

也就是说上面这个函数是用于随机生成 64 位十六进制字符

更为科学的解释:
> ⽣成密钥的第⼀步也是最重要的⼀步， 是要找到⾜够安全的熵源， 即随机性来源。 ⽣成⼀个⽐特币私钥在本质上与 “在 1 到 2^256 之间选⼀个数字” ⽆异。 只要选取的结果是不可预测或不可重复的， 那么选取数字的具体⽅法并不重要。 ⽐特币软件使⽤操作系统底层的随机数⽣成器来产⽣ 256 位的熵（ 随机性） 。 通常情况下， 操作系统随机数⽣成器由⼈⼯的随机源进⾏初始化， 也可能需要通过⼏秒钟内不停晃动⿏标等⽅式进⾏初始化。 对于真正的偏执狂， 可以使⽤掷骰⼦的⽅法， 并⽤铅笔和纸记录。

[参考链接](http://pangjiuzala.github.io/2016/03/03/Bitcoin%E5%8A%A0%E5%AF%86%E6%8A%80%E6%9C%AF%E4%B9%8B%E6%A4%AD%E5%9C%86%E6%9B%B2%E7%BA%BF%E5%AF%86%E7%A0%81%E5%AD%A6/)
http://pangjiuzala.github.io/2016/03/03/Bitcoin%E5%8A%A0%E5%AF%86%E6%8A%80%E6%9C%AF%E4%B9%8B%E6%A4%AD%E5%9C%86%E6%9B%B2%E7%BA%BF%E5%AF%86%E7%A0%81%E5%AD%A6/

```
/* generate a public key from a private key */
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
```
这是使用了椭圆曲线加密算法, 具体的介绍见 [这里](http://pangjiuzala.github.io/2016/03/03/Bitcoin%E5%8A%A0%E5%AF%86%E6%8A%80%E6%9C%AF%E4%B9%8B%E6%A4%AD%E5%9C%86%E6%9B%B2%E7%BA%BF%E5%AF%86%E7%A0%81%E5%AD%A6/)
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

该函数作用:
1 - Take the corresponding public key generated with it (65 bytes, 1 byte 0x04, 32 bytes corresponding to X coordinate, 32 bytes corresponding to Y coordinate)

```
/* provide a public key and return address */
	coinjs.pubkey2address = function(h, byte){
		var r = ripemd160(Crypto.SHA256(Crypto.util.hexToBytes(h), {asBytes: true}));
		r.unshift(byte || coinjs.pub);
		var hash = Crypto.SHA256(Crypto.SHA256(r, {asBytes: true}), {asBytes: true});
		var checksum = hash.slice(0, 4);
		return coinjs.base58encode(r.concat(checksum));
	}
```
该函数作用:
2 - Perform SHA-256 hashing on the public key
3 - Perform RIPEMD-160 hashing on the result of SHA-256
4 - Add version byte in front of RIPEMD-160 hash (0x00 for Main Network)
(note that below steps are the Base58Check encoding, which has multiple library options available implementing it)
```
/* provide a privkey and return an WIF  */
	coinjs.privkey2wif = function(h){
		var r = Crypto.util.hexToBytes(h);
		if(coinjs.compressed==true){
			r.push(0x01);
		}
		r.unshift(coinjs.priv);
		var hash = Crypto.SHA256(Crypto.SHA256(r, {asBytes: true}), {asBytes: true});
		var checksum = hash.slice(0, 4);
		return coinjs.base58encode(r.concat(checksum));
	}
```
WIF 全称 (Wallet Import Format), 作用是导出导入密钥, 方便备份迁移.
Note: The WIF (Wallet Import Format) includes information about the network and if the associated public key is compressed or uncompressed (thus the same bitcoin address will be generated by using this format).
## 功能 2: 多方签名

(typeof Crypto=="undefined"||!Crypto.util)&&function(){
  var f=window.Crypto={},
    l=f.util={rotl:function(b,a){return b<<a|b>>>32-a},
      rotr:function(b,a){return b<<32-a|b>>>a},
      endian:function(b){
        if(b.constructor==Number)
          return l.rotl(b,8)&16711935|l.rotl(b,24)&4278255360;
        for(var a=0;a<b.length;a++)
          b[a]=l.endian(b[a]);
        return b},
        randomBytes:function(b){
          for(var a=[];b>0;b--)
            a.push(Math.floor(Math.random()*256));
          return a},
        bytesToWords:function(b){
          for(var a=[],c=0,d=0;c<b.length;c++,d+=8)
            a[d>>>5]|=(b[c]&255)<<24-d%32;
          return a},
        wordsToBytes:function(b){
          for(var a=[],c=0;c<b.length*32;c+=8)
            a.push(b[c>>>5]>>>24-c%32&255);
          return a},
        bytesToHex:function(b){
          for(var a=[],c=0;c<b.length;c++)
            a.push((b[c]>>>4).toString(16)),a.push((b[c]&15).toString(16));
          return a.join("")},
        hexToBytes:function(b){
          for(var a=[],c=0;c<b.length;c+=2)
            a.push(parseInt(b.substr(c,2),16));
          return a},
        bytesToBase64:function(b){
          for(var a=[],c=0;c<b.length;c+=3)
            for(var d=b[c]<<16|b[c+1]<<8|b[c+2],q=0;q<4;q++)
              c*8+q*6<=b.length*8?a.push("ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/".charAt(d>>>6*(3-q)&63)):a.push("=");
              return a.join("")},
        base64ToBytes:function(b){
          for(var b=b.replace(/[^A-Z0-9+\/]/ig,""),a=[],c=0,d=0;c<b.length;d=++c%4)
            d!=0&&a.push(("ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/".indexOf(b.charAt(c-1))&Math.pow(2,-2*d+8)-1)<<d*2|"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/".indexOf(b.charAt(c))>>>6-d*2);
            return a}
    },
    f=f.charenc={};
    f.UTF8={stringToBytes:function(b){
      return i.stringToBytes(unescape(encodeURIComponent(b)))},
      bytesToString:function(b){
        return decodeURIComponent(escape(i.bytesToString(b)))}};
    var i=f.Binary={stringToBytes:function(b){
      for(var a=[],c=0;c<b.length;c++)
        a.push(b.charCodeAt(c)&255);
        return a},
                  bytesToString:function(b){
                    for(var a=[],c=0;c<b.length;c++)
                      a.push(String.fromCharCode(b[c]));
                    return a.join("")}}}();
(function(){
  var f=Crypto,
  l=f.util,
  i=f.charenc,
  b=i.UTF8,
  a=i.Binary,
  c=[1116352408,1899447441,3049323471,3921009573,961987163,1508970993,2453635748,2870763221,3624381080,310598401,607225278,1426881987,1925078388,2162078206,2614888103,3248222580,3835390401,4022224774,264347078,604807628,770255983,1249150122,1555081692,1996064986,2554220882,2821834349,2952996808,3210313671,3336571891,3584528711,113926993,338241895,666307205,773529912,1294757372,1396182291,1695183700,1986661051,2177026350,2456956037,2730485921,
2820302411,3259730800,3345764771,3516065817,3600352804,4094571909,275423344,430227734,506948616,659060556,883997877,958139571,1322822218,1537002063,1747873779,1955562222,2024104815,2227730452,2361852424,2428436474,2756734187,3204031479,3329325298],
d=f.SHA256=function(b,c){
  var e=l.wordsToBytes(d._sha256(b));
    return c&&c.asBytes?e:c&&c.asString?a.bytesToString(e):l.bytesToHex(e)};
    d._sha256=function(a){
      a.constructor==String&&(a=b.stringToBytes(a));
      var d=l.bytesToWords(a),e=a.length*8,a=[1779033703,3144134277,
1013904242,2773480762,1359893119,2600822924,528734635,1541459225],f=[],m,n,i,h,o,p,r,s,g,k,j;
d[e>>5]|=128<<24-e%32;
d[(e+64>>9<<4)+15]=e;
for(s=0;s<d.length;s+=16){
  e=a[0];
  m=a[1];
  n=a[2];
  i=a[3];
  h=a[4];
  o=a[5];
  p=a[6];
  r=a[7];
  for(g=0;g<64;g++){
    g<16?f[g]=d[g+s]:(k=f[g-15],j=f[g-2],f[g]=((k<<25|k>>>7)^(k<<14|k>>>18)^k>>>3)+(f[g-7]>>>0)+((j<<15|j>>>17)^(j<<13|j>>>19)^j>>>10)+(f[g-16]>>>0));
    j=e&m^e&n^m&n;
    var t=(e<<30|e>>>2)^(e<<19|e>>>13)^(e<<10|e>>>22);
    k=(r>>>0)+((h<<26|h>>>6)^(h<<21|h>>>11)^(h<<7|h>>>25))+(h&o^~h&p)+c[g]+(f[g]>>>0);
    j=t+j;
    r=p;
    p=o;
    o=h;
    h=i+k>>>0;
    i=n;
    n=m;
    m=e;
    e=k+j>>>0}
      a[0]+=e;
    a[1]+=m;
    a[2]+=n;
    a[3]+=i;
    a[4]+=h;
    a[5]+=o;
    a[6]+=p;
    a[7]+=r}
    return a};
    d._blocksize=16;d._digestsize=32})();
