# openssl

使用 openssl 的库函数生成签名

文件列表:
```
base58.h
common.h
ec.h
endian.h
ex-address.c
ex-base58.c
ex-ec-keypair.c
ex-ecdsa-sign.c
ex-ecdsa-verify.c
ex-fixed-strings.c
ex-hashes.c
ex-integers.c
ex-tx-build.c
ex-tx-pack.c
ex-tx-sign.c
ex-varints.c
ex-wif.c
tx.h
varint.h
```
base58.h
提供两个函数, base58 和 base58check
base58 负责编码, base58check 负责生成摘要和校验码

common.h
bbp_print_hex: 打印输出十六进制格式
bbp_hex2byte: 将单个字符转化为 byte
bbp_parse_hex: 将字符串通过调用 bbp_hex2byte 转化为十六进制, 每两个字符为一组, 转化成一个十六进制单位
bbp_alloc_hex: 和 bbp_parse_hex 功能一样

ec.h
bbp_ec_new_keypair: 创建公私钥对
bbp_ec_new_pubkey: 通过 pubkey 的 byte 形式转化为 Key 形式

endian.h
bbp_host_endian: 判断宿主机是大端还是小端
bbp_swap16: 将 16 bit 左右 8 bit 交换
bbp_swap32: 将 32 bit 分成 4 份, 0,1,2,3; 交换后变成 3,2,1,0
bbp_swap64: 同上
bbp_eint16:
bbp_eint32:
bbp_eint64: 判断宿主机是不是等于传入的参数 e(表示大端小端), 是则返回参数, 不是则调用 swap 交换
bbp_reverse:


目前完成功能:
除了本来已完成的功能外
多方签名钱包地址和redeemScript的构建
