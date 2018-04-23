
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

https://omarmetwally.blog/2017/07/25/how-to-create-a-private-ethereum-network/
