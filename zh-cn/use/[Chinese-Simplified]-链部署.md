用户可以通过脚本一键启动单个节点或者添加多个节点组网进行共识。

1. 快速演示: 启动单节点区块链

```shell
# 1. 生成描述区块链信息的genesis文件
cd ${WORKSPACE}/scripts
./platonectl.sh setupgen -n 0 --ip 127.0.0.1 --p2p_port 16791 --auto true  

# 2. 初始化第一个节点(node 0)
./platonectl.sh init -n 0 --ip 127.0.0.1 --rpc_port 6791 --p2p_port 16791 \
--ws_port 26791 --auto "true"

# 3. 开启节点 (node 0)
./platonectl.sh start -n 0

# 4. 部署系统合约
./platonectl.sh deploysys -n 0  --auto "true"
```

2. 快速演示: 将其他节点添加到链中

```shell
# 1. 初始化node-1
./platonectl.sh init -n 1 --ip 127.0.0.1 --rpc_port 6792 --p2p_port 16792 \
--ws_port 26792 --auto "true"

# 2. 启动node-1
./platonectl.sh start -n 1

# 3. 将节点添加到已存在的区块链中
./platonectl.sh addnode -n 1 

# 4. 将node-1更新为共识节点
./platonectl.sh updatesys -n 1
```

3. 关闭节点

```shell
./platonectl.sh stop -n 0 
```

4. 连接到console

```
./platonectl.sh console -n 0
```

5. 一键启动

```shell
# 快速启动单节点
./platonectl.sh one

# 在一台机器上快速启动四节点区块链
./platonectl.sh four
```
