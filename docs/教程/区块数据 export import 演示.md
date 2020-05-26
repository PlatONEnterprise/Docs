# 区块数据 export import 演示

[TOC]

## 1. 起四个节点的链

```shell
./platonectl.sh four

# 此时经过部署一系列系统合约，区块高度已经到达15
```

## 2. 停节点，并export指定区块的数据

```shell
# 停掉进程
./platonectl.sh stop -a

# export指定区块数据，比如此处导出0~14区块数据
cd ../bin

./platone export --datadir ../data/node-0/  block-0-14.rlp 0 14
```

## 3. 清掉四个节点的数据目录，并根据已有的genesis初始化链

```shell
rm -rf  ../data/node-*/platone/*

./platone init --datadir ../data/node-0 ../conf/genesis.json
./platone init --datadir ../data/node-1 ../conf/genesis.json
./platone init --datadir ../data/node-2 ../conf/genesis.json
./platone init --datadir ../data/node-3 ../conf/genesis.json
```

## 4. 导入第二步生成的区块数据

```shell
# 给节点0导入数据
./platone import --datadir ../data/node-0 block-0-14.rlp

# 然后启动节点0
cd ../scripts
./platonectl.sh start -n 0

# 此时观察log会发现节点0的区块高度已经成为14了，其他节点可以启动，然后跟节点0连接，同步其数据，最终整个区块链高度都是14了
```
