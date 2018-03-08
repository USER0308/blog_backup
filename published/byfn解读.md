# byfn.sh

先判断操作系统是 osx 还是 Linux , x86_64 还是 amd64

设置默认值: CLI_TIMEOUT , CHANNEL_NAME , COMPOSE_FILE = docker-compose-cli.yaml

判断传入的命令( command ),

* 如果是 h 或其他,打印帮助信息,退出
* 如果是 m ,则设置 MODE 为传入参数
* 如果是 c ,则设置 CHANNEL_NAME 为传入参数
* 如果是 t ,则设置CLI_TIMEOUT 为传入参数

根据 MODE 的取值
* 为 up ,调用 networkup 函数
* 为 down ,调用 networkdown 函数
* 为 generate ,调用 generateCerts , replacePrivatekey , generateChannelArtifacts 函数
* 为 restart,调用 networkdown 和 networkup
* 为其他,打印帮助信息,退出

##  generate

###  generateCerts 函数:

  查看 cryptogen 所在位置,(如果是自动搭建网络,需要把 cryptogen 所在目录添加到 PATH 路径中)
  若没找到,打印出错信息,退出
  找到则执行

    cryptogen generate --config=./crypto-config.yaml

  若上述命令执行出错,则退出

### replacePrivatekey
  查看操作系统是 osx 还是 Linux ,根据操作系统设置 OPTS 参数
  把 docker-compose-e2e-template.yaml 复制一份,改名为 docker-compose-e2e.yaml
  保存当前路径为 CURRENT_DIR
  切换路径到 crypto-config/peerOrganizations/org1.example.com/ca/
  把当前目录下的以 \_sk 结尾的文件名保存在 PRIV_KEY 变量中
  切换回 CURRENT_DIR , 把 docker-compose-e2e.yaml 中的 CA/\_PRIVATE_KEY 替换为 PRIV_KEY
  替换的命令与操作系统平台有关,用到上面的 OPTS 参数
  同样,切换到 crypto-config/peerOrganizations/org2.example.com/ca/ 目录下,把当前目录下的 \*\_sk 文件名保存在 PRIV_KEY 变量中,切换回 CURRENT_DIR,把docker-compose-e2e.yaml中 的 CA/\_PRIVATE_KEY 替换为 PRIV_KEY
  切换回 CURRENT_DIR ,若是操作系统为 osx ,还要删除当前目录下的 docker-compose-e2e.yamlt 文件

### generateChannelArtifacts

  查找 configtxgen 所在位置,(提前加入 PATH )
  若找不到,报错退出
  生成 genesis.block
  (事实上应为 order.genesis.block ,由于某些原因,至少到目前为止还是,这个 block 不能被命名为 orderer.genesis.block ,否则  orderer 将无法启动)
  执行

    configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

  如果上述命令执行失败,报错退出

  生成 channel configuration transaction:channel.tx
  执行

    configtxgen -profile TwoOrgsChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

  失败则报错退出

  生成 anchor peer update for Org1MSP
  执行

    configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -ChannelID $CHANNEL_NAME -asOrg Org1MSP

  失败则报错退出

  生成 anchor peer update for Org2MSP
  执行

    configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -ChannelID $CHANNEL_NAME -asOrg Org2MSP

  失败则报错退出

## networkup

  判断是否存在 crypto-config 目录
    没有的话,调用 generate 的三个函数
  设置 CHANNEL_NAME,TIMEOUT

    docker compose -f docker-compose-cli.yaml up -d

  把标准错误输出重定向到标准输出
  如果出错,打印日志, docker logs -f cli ,退出
  没出错,打印日志 docker logs -f cli

## networkdown

    docker compose docker-compose-cli.yaml down

  如果 MODE 是 restart 的话,
    调用 clearContainers , removeUnwantedImages 函数

    rm -rf channel-artifacts/\*.block channel-artifacts/\*.tx crypto-config
    rm -f docker-compose-e2e.yaml

  clearContainers
    列出所有 containerID
    docker rm -f ID
  removeUnwantedImages
    找出含有 dev,none,test_vp,peer0-9 的 ImageID
    docker rmi -f ID

# script.sh 脚本

docker-compose-cli.yaml up 之后执行 command ,即调用./script/script.sh脚本

## 首先创建channel:createChannel()

  调用 setGlobals 函数,传入参数 0 (设置相关环境变量,使得 cli 操控的是 peer0.Org1 )
  判断 \$CORE\_PEE\R_TLS\_ENABLED 是否为空或者为 false
    是则调用

      peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx

  把输出写入日志
    否则调用

      peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA

  两者差别在于是否使用了 TLS 传输层安全协议
  把上面命令执行结果赋值给 res ,输出 log 日志,把 res 作为参数传递给 verifyResult 函数

## 把所有节点都加入 channel:joinChannel()

  利用循环,把 0 , 1 , 2 , 3 (代表 peer0.Org1 , peer1.Org1 , peer0.Org2 , peer1.Org2 )作为参数 ch 传递给 setGlobals 函数,调用 joinWithRetry 函数,传入 ch 参数
    joinWithRetry 函数中 调用

      peer channel join -b $CHANNEL_NAME.block

  把输出写入日志,把执行结果赋值为 res ,输出日志,判断如果执行结果不为 0 (即执行失败)且计数变量 COUNTER 小于 MAX_RETRY 常量(等于 5 )
   COUNTER 加一, sleep 两秒后递归调用自身
  否则,设置 COUNTER 为 1 ,
  把执行结果赋值 res ,传递给 varifyResult 函数
## 把 peer0.Org1 节点更新为锚结点: updateAnchorPeers 0
  把传入的参数赋值为 PEER , 把 PEER 传递给 setGlobals ,判断 \$CORE_PEER_TLS_ENABLED 是否为空或者为 false
    是则调用

      peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx

  把输出写入日志
    否则调用

      peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA

  把输出写入日志
    两者区别在于是否使用了 TLS
  把上面命令执行结果赋值给 res ,输出 log 日志,把 res 作为参数传递给 verifyResult 函数

## 把 peer0.Org2 节点更新为锚结点: updateAnchorPeers 2

  同上

## 在 peer0.Org1 节点上安装 chaincode : installChaincode 0

  把传入的参数赋值为 PEER , 把 PEER 传递给 setGlobals ,调用

    peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02

  把输出写入日志
  把上面命令执行结果赋值给 res ,输出 log 日志,把 res 作为参数传递给 verifyResult 函数

## 在 peer0.Org2 节点上安装 chaincode : installChaincode 2

  同上

## 在 peer0.Org2 节点上实例化 chaincode : instantiateChaincode 2

  把传入的参数赋值为 PEER , 把 PEER 传递给 setGlobals ,判断 \$CORE_PEER_TLS_ENABLED 是否为空或者为 false
    是则调用

      peer chaincode instantiate -o orderer.example.com:7050 -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR	('Org1MSP.member','Org2MSP.member')"

  把输出写入日志
  否则调用

    peer chaincode instantiate -o orderer.example.com:7050 -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR	('Org1MSP.member','Org2MSP.member')

  把输出写入日志
  把上面命令执行结果赋值给 res ,输出 log 日志,把 res 作为参数传递给 verifyResult 函数
## 在 peer0.Org1 节点上查询,看查询结果是否等于 100 : chaincodeQuery 0 100
  把传入的参数赋值为 PEER , 把 PEER 传递给 setGlobals ,设置临时变量 rc 用于 flag 作用,设置临时变量 starttime 为当前时间,循环检查当前时间减去开始时间是否小于超时时间且 rc 变量不为 0 ,则执行:
    睡眠 3 秒,
    执行

      peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

  把结果写入 log 日志
  检测上一句命令执行结果是否为 0 ,若是则把 log 日志中的 Query Result 赋值为 VALUE ,
  检查 VALUE 是否等于传入的第二个参数,若是则把 rc 变量改为 0
  循环结束
  输出 log 日志,若 rc 为 0 ,则输出查询成功,否则输出失败,退出

## 在 peer0.Org1 节点上转账: chaincodeInvoke 0

  把传入的参数赋值为 PEER , 把 PEER 传递给 setGlobals, 判断 \$CORE_PEER_TLS_ENABLED 是否为空或者为 false
  是则调用

    peer chaincode invoke -o orderer.example.com:7050 -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

  把输出写入到日志
  否则调用

    peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

  把输出写入日志
  把上面命令执行结果赋值给 res ,输出 log 日志,把 res 作为参数传递给 verifyResult 函数

## 在 peer1.Org2 节点上安装 chaincode : installChaincode 3

  同上

## 在 peer1.Org2 节点上查询,看查询结果是否等于 90 : chaincodeQuery 3 90

  同上

### setGlobals 函数

  如果传入的参数是 0 或 1 ,
    设置环境变量:

      CORE_PEER_LOCALMSPID="Org1MSP"
      CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
      CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

  如果传入的是 0 ,
  设置环境变量:

    CORE_PEER_ADDRESS=peer0.org1.example.com:7051

  如果传入的是 1 ,
  设置环境变量:

    CORE_PEER_ADDRESS=peer1.org1.example.com:7051

  如果传入的不是 0 或 1 (即 2 或 3 ),
    设置环境变量:

    CORE_PEER_LOCALMSPID="Org2MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp

  如果传入的是 2 ,
  设置环境变量:

    CORE_PEER_ADDRESS=peer0.org2.example.com:7051

  如果传入的是 3 ,
  设置环境变量:

    CORE_PEER_ADDRESS=peer1.org2.example.com:7051

### verifyResult 函数

  如果传入参数不等于 0 ,则输出错误信息
