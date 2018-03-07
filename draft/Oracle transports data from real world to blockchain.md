# 链下数据上链技术研究

    区块链还是一个早期的技术, 还无法完全通过区块链打造一个闭环的应用系统, 都会涉及到跟链下系统对接的问题. 但是如何保证链下数据放到区块链上之后的真实性, 还是个问题. 本课题研究数据如何通过预言机 (Oracle) 上链, 以及上链数据验证技术.

智能合约中需要真实世界系统信息的场景:

* 航空公司与用户签订智能合约, 如果航班延误, 航空公司将为用户支付一定金额. 智能合约需要知道真实世界中航班是否延误.

* 智能合约需要产生真随机数, 必须要从 https://www.random.org 获取, 区块链智能合约无法单独完成发起 http(s) 请求

通过 Oracle 预言机数据上链

### Oracle 是什么?

    In classical antiquity, an oracle was a person or agency considered to provide wise and insightful counsel or prophetic predictions or precognition of the future, inspired by the gods. As such it is a form of divination.

    -- from https://en.wikipedia.org/wiki/Oracle

简而言之, 预言机是一种可信任实体, 具有不可篡改, 服务稳定, 可审计等特点. 它 (目前) 是一个第三方机构, 也是中心化机构. 对于区块链中智能合约提出的问题, 预言机可以给出答案, 并且确保答案在获取和传输的过程中未经篡改.

技术研究:

目前已使用了 Oracle 技术的项目有 amoveo,github 地址 (https://github.com/zack-bitcoin/amoveo), Aethernity,github 地址 (https://github.com/aeternity),amoveo 的主要维护人员是 Aethernity 的主要开发人员, 因不满薪资而离职, 自己重开 amoveo 项目. 以上两个项目都成熟地运用了 Oracle 预言机. 而这两个项目中使用的 Oracle 预言机实际上是 Oraclize 提供的服务实现其功能. Oraclize github 地址 (https://github.com/oraclize)

The solution developed by Oraclize is instead to demonstrate that the data fetched from the original data-source is genuine and untampered. This is accomplished by accompanying the returned data together with a document called authenticity proof. The authenticity proofs can build upon different technologies such as auditable virtual machines and Trusted Execution Environments.

This solutions elegantly solves the Oracle Problem:

Blockchain Application’s developers and the users of such applications don’t have to trust Oraclize; the security model is maintained.
Data providers don’t have to modify their services in order to be compatible with blockchain protocols. Smart contracts can directly access data from Web sites or APIs.

Data Source Types
* URL: enables the access to any webpage or HTTP API endpoint
* WolframAlpha: enables native access to WolframAlpha computational engine
* IPFS: provides access to any content stored on an IPFS file
* random: provides untampered random bytes coming from a secure application running on a Ledger Nano S.
* computation: provides the result of arbitrary computation
* nested: enables the combination of different types of data source or multiple requests using the same data source, and it returns an unique result
* identity: it returns the query
* decrypt: it decrypts a string encrypted to the Oraclize private key

!!
If Oraclize is unable to generate an authenticity proof for technical reasons, it will return in most cases the result without the requested proof. It is up to the developer to decide how to handle this case in their application: Oraclize recommends to discards the result and create a new query.

The interaction between Oraclize and an Ethereum smart contract is asynchronous. Any request for data is composed of two steps:

Firstly, in the most common case, a transaction executing a function of a smart contract is broadcasted by an user. The function contains a special instruction which manifest to Oraclize, who is constantly monitoring the Ethereum blockchain for such instruction, a request for data.
Secondly, according to the parameters of such request, Oraclize will fetch or compute a result, build, sign and broadcast the transaction carrying the result. In the default configuration, such transaction will execute the __callback function which should be placed in the smart contract by its developer: for this reason, this transaction is referred in the documentation as the Oraclize callback transaction.


install Ethereum

https://omarmetwally.blog/2017/07/25/how-to-create-a-private-ethereum-network/

deploy oracle smart_contract

https://ethereum.stackexchange.com/questions/11383/oracle-oraclize-it-with-truffle-and-testrpc/11389#11389

write compile build oracle code

http://dapps.oraclize.it/browser-solidity/


my code

init geth
```
geth --datadir=~/Projects/ethereum/smart_contract/data init ~/Projects/ethereum/smart_contract/genesis.json
```
create node
```
bootnode --genkey=boot.key
```
booting node
```
bootnode --nodekey=boot.key
```
connect to the node
```
geth --datadir=~/Projects/ethereum/smart_contract/data --bootnodes=enode://0b197a70e19274ffdba78a82dfcf3bd055297180ced6bb54ffc339411c3f7c7cd04df52d58936f7b48aa171db5d413aaf849f2d6bbe7af83be2b7554ccfb3c59@[::]:30301
```
or the one below(maybe correct)
```
geth --datadir=~/Projects/ethereum/smart_contract/data --bootnodes=enode://0b197a70e19274ffdba78a82dfcf3bd055297180ced6bb54ffc339411c3f7c7cd04df52d58936f7b48aa171db5d413aaf849f2d6bbe7af83be2b7554ccfb3c59@127.0.0.1:30301
```
start the console
```
geth --identity "private chain" --rpc --rpcport "8545" --rpccorsdomain "*" --rpcapi "db,eth,net,web3" --datadir "/home/user0308/Projects/ethereum/smart_contract/data" --nodiscover --networkid 15 console
```
or
```
geth attach /home/user0308/Projects/ethereum/smart_contract/data/geth.ipc
```

mining

```
geth --datadir=~/Projects/ethereum/smart_contract/data --mine --minerthreads=1 --etherbase=0xcb1a8b440b28ec06c5e76474350603583a42c668
```

start bridge
```
node bridge -H 127.0.0.1:8545 -a 1
```

gitter message
ASK: @oraclize-support Will the ethereum-bridge solve it so I can run the RandomExample on a private net?

ANSWER:@tsuriyathep noy really: the random ds proof can be valid on public chains only, if you need to test it privately, disable temporarily the proof verification (but don't forget to put it back before going to production!)

ASK:@oraclize-support I would really like to get the RandomSample working, is it possible to get a short tutorial or add more detail to the readme file, so we can get instructions on how to get it up and running in any kind of development environment?

ANSWER:@tsuriyathep we will do that, thanks for the suggestion. In the meantime, let us help you here! It should work out of the box in any context when you don't verify the authenticity proof, did you get it working in this way?
