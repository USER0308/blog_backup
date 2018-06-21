编译crypto++库
源码从官网https://cryptopp.com/下载
版本:cryptopp6.1.0
编译参考链接:https://www.cryptopp.com/wiki/Linux

编译成功后会在/usr/local/include/cryptopp路径

#应该也可以不编译,使用相对路径引入需要的文件即可(待测试)

运行测试参考makefile文件
=============
CC=g++
CFLAGS=--std c++11 

test_multisig: test_multisig.cpp
	$(CC) $(CFLAGS) -o test_multisig.out test_multisig.cpp

test_sign: test_sign.cpp
	$(CC) $(CFLAGS) -o test_sign.out test_sign.cpp -lcryptopp -lpthread

test_ecc: test_ecc.cpp ./crypto/Schnorr.cpp
	$(CC) test_ecc.cpp ./crypto/Schnorr.cpp -o test_ecc.out -lcryptopp -lpthread

test_match: test_match.cpp
	$(CC) test_match.cpp -o test_match.out -lcryptopp -lpthread

test_verify: test_verify.cpp
	$(CC) test_verify.cpp -o test_verify.out -lcryptopp -lpthread

test_all: test_all.cpp
	$(CC) test_all.cpp -o test_all.out -lcryptopp -lpthread

run:
		make $(ARGS) && ./$(ARGS).out
clean:
	rm -f *.out *.o
=============
用的是g++编译器,支持C++11
编译:命令行中执行make test_XXX
编译且运行:命令行中执行make run ARGS=test_XXX

一般来说,在编译命令中 -lcryptopp参数需要放在末尾

