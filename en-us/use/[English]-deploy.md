Users could run script to start one node or permit other nodes to join the network:

1. Start one node:

```shell
# 1. generate genisis state 
cd PlatONE-Workspace/chain/PlatONE_linux/scripts
./platonectl.sh setupgen -n 0 --ip 127.0.0.1 --p2p_port 16791 --auto true  

# 2. initialize the node(node 0)
./platonectl.sh init -n 0 --ip 127.0.0.1 --rpc_port 6791 --p2p_port 16791 \
--ws_port 26791 --auto "true"

#  start the node (node 0)
./platonectl.sh start -n 0

# 4. deploy system contract 
./platonectl.sh deploysys -n 0  

```

2.  Add new node

```shell
# 1. initailize new node
./platonectl.sh init -n 1 --ip 127.0.0.1 --rpc_port 6792 --p2p_port 16792 \
--ws_port 26792 --auto "true"

# 2. start new node
./platonectl.sh start -n 1

#  register new node
./platonectl.sh addnode -n 1 

# 4. update new node to consensus node
./platonectl.sh updatesys -n 1

```

3. stop the node

```shell
./platonectl.sh stop -n 0 
```

4. attach the console of the node

```
./platonectl.sh console -n 0
```

5. quickly start

```shell
# start one node
./platonectl.sh one

# start four nodes in one machine
./platonectl.sh four
```



