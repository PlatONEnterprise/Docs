# PlatONE Script Deployment Tutorial

Section 1 of this document describes a simple case demonstration of deploying PlatONE using scripts; Chapter 2 describes the functions and usage of various commands for scripts; Section 3 details the command line parameters for each command of the script. The above three chapters describe how PlatONE deploys scripts on a single machine. Section 4 demonstrates how PlatONE deploys scripts on multiple machines. As for Sections 5, 6, and 7, PlatONE's recommendations for maintenance, such as backup, log cleanup strategies, and operational status checks, are given.

## 1. Case Demo (for simple test environment)

If it is the first time for you to start PlatONE blockchain, the best choice is to start one node and then add other nodes to the chain. In part 1.1 and 1.2 we give a brief introduction to start the chain and add new nodes to the chain. Besides, more details of deploying the blockchain can be seen in part 2. 

```bash
# get PlatONE source code
git clone --recursive https://github.com/PlatONEnterprise/PlatONE-Go.git
export WORKSPACE=${PWD}/PlatONE-Go/release/linux
# Compile PlatONE
cd PlatONE-Go; make all; cd ..
# If the compilation fails, make sure you have installed all the software in 1.3.1.Then try again.
```

Now you can put the `PlatONE-Go/release/linux` folder on the rest of the server, and you can deploy and start the chain.

### 1.1. Quick Demo: Start a One-Node Blockchain

```shell
# 1. Generate the genesis file which describes the information of the blockchain
cd ${WORKSPACE}/scripts
./platonectl.sh setupgen -n 0 --ip 127.0.0.1 --p2p_port 16791 --auto true  

# 2. Init the first node (node 0)
./platonectl.sh init -n 0 --ip 127.0.0.1 --rpc_port 6791 --p2p_port 16791 \
--ws_port 26791 --auto "true"

# 3. Start the node (node 0)
./platonectl.sh start -n 0

# 4. Deploy system contracts
./platonectl.sh deploysys -n 0 --auto "true"
```

### 1.2. Quick Demo: Add Other Nodes to the Chain

```shell
# 1. Init  node-1
./platonectl.sh init -n 1 --ip 127.0.0.1 --rpc_port 6792 --p2p_port 16792 \
--ws_port 26792 --auto "true"

# 2. start  node-1
./platonectl.sh start -n 1

# 3. add node to the already existed blockchain 
./platonectl.sh addnode -n 1 

# 4. update node-1 to consensus node
./platonectl.sh updatesys -n 1 
```

## 2. Script Function Demo

### 2.1. Environment Preparation

Before compiling, you need to download the following software first:

* cmake 3.10+
* go 1.11+
* gcc 7.3+

### 2.2. Setup genesis.json

When you start a blockchain, you need create the genesis.json file firstly.

```bash
cd ${WORKSPACE}/scripts
./platonectl.sh setupgen -n 0 --ip 127.0.0.1 --p2p_port 16791 --auto true  
```

* auto=false: Will prompt for the option to compile the system contract

you can also use values by default.

```bash
./platonectl.sh setupgen
```

### 2.3. Init Node

```bash
./platonectl.sh init -n 0 --ip 127.0.0.1 --rpc_port 6791 --p2p_port 16791 \
												--ws_port 26791 --auto "true"
```

you can also use values by default.

```bash
./platonectl.sh init
```

### 2.4. Start Node

```bash
./platonectl.sh start -n 0
```

When starting the node, you can also specify the path to the log folder, the size of the log file partition, specify additional command line parameters when the platone starts, etc. (Note: The path connector '/' needs to be escaped. The value of the parameter option must be quoted)

```bash
./platonectl.sh start -n 0 -d '.\/logs\/platone' -s 65535 -e '--verbosity 3 --debug'
```

The log folder contains the logs executed by wasm and the logs run by platone. Over time, the log files will accumulate and it is recommended to mount or periodically delete them.

### 2.5. Deploy System Contracts

```bash
./platonectl.sh deploysys -n 0 --auto true 
```

### 2.6. Add Node

You may want to add some nodes to PlatONE network, you must init this new node first.

- Init a new node(node 1):

```bash
./platonectl.sh init -n 1 --ip 127.0.0.1 --rpc_port 6792 --p2p_port 16792 \
												--ws_port 26792 --auto "true"
```

- Add the new node:

```bash
./platonectl.sh addnode -n 1
```

- you can also add the remote node:

```bash
./platonectl.sh addnode -n 10 --p2p_port 3333 --rpc_port 4444 --ip xxx.xxx.xxx.xxx --pubkey xxxxx
```

### 2.7. Update Node

Upgrade the node to be the consensus node.

```bash
./platonectl.sh updatesys -n 1 
```

you can also update the node status:

```bash
./platonectl.sh updatesys -n 1 -c '{"status":2}'
```

### 2.8. Create Account

Create account for node:

```bash
./platonectl.sh createacc -n 1
```

### 2.9. Unlock Account

Unlock the node account. Make sure that the node has been started before you unlock the node account.

```bash
./platonectl.sh unlock -n 1
```

### 2.10. Connect to Console

you can open an  interactive JavaScript environment :

```bash
./platonectl.sh console -n 0
```

### 2.11. get all nodes

you can get all nodes from system contract 

```bash
./platonectl.sh get
```

### 2.12. view node info

The following command can be used to get the node status and information:

```bash
./platonectl.sh status -n 0
```

console:

```console
disable -> node_id:  0
          node info:
                  node.address: f92f1c1469d1a8c38964c63d62d3167842ce70cd
                  node.ip: 127.0.0.1
                  node.rpc_port: 6791
                  node.p2p_port: 16791
                  node.ws_port: 26791
                  node.pubkey: 9afcb45d2725059a5a6a7f379d6404b6c914b02dd3e7a33d29c87b3f28f8f63b78ffe2736a3cb52ae45bbf57471d438eac71ddcc0bdfbaa56d65e59d457159b2
```

### 2.13. stop and clear

you can stop the node and clear node data:

```bash
./platonectl.sh stop -n 0 
./platonectl.sh clear -n 0
```

It can also be simplified as follows: (clearance includes pause function):

```bash
./platonectl.sh clear -n 0
```

### 2.14. One-click start (for simple test environment)

This section is to describe how to start one node or start four nodes in the same machine quickly.

**Start one-node**

quickly start one-node blockchain:

```bash
./platonectl.sh one
```

**Start four-nodes**

quickly start  four-nodes blockchain on one machine:

```bash
./platonectl.sh four
```

## 3. Command Details

This section gives a detailed introduction to command.

```console
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
        init                             initialize node. please setup genesis first
        one                              start a node completely
                                         default account password: 0
        four                             start four node completely
                                         default account password: 0
        start                            try to start the specified node
        stop                             try to stop the specified node
        restart                          try to restart the specified node
        console                          start an interactive JavaScript environment
        deploysys                        deploy the system contract
        updatesys                        normal node update to consensus node
        addnode                          add normal node to system contract
        clear                            clear all nodes data
        unlock                           unlock node account
        get                              display all nodes in the system contract
        setupgen                         create the genesis.json and compile system contract
        status                           show all node status
        createacc                        create account
```

### 3.1. init

example: 

```bash
./platonectl.sh init -h
```

console: 

```bash
        init OPTIONS
            --nodeid, -n                 set node id (default=0)
            --ip                         set node ip (default=127.0.0.1)
            --rpc_port                   set node rpc port (default=6791)
            --p2p_port                   set node p2p port (default=16791)
            --ws_port                    set node ws port (default=26791)
            --auto                       auto=true: will no prompt to create
                                         the node key and init (default: false)
            --help, -h                   show help
        example: platonectl.sh init -n 1
                                    --ip 127.0.0.1
                                    --rpc_port 6790
                                    --p2p_port 16790
                                    --ws_port 26790
                                    --auto "true"
                 or:
                     platonectl.sh init
```

### 3.2. one

example:

```bash
./platonectl.sh one 
```

This case will do:

- create the genesis.json

- init the first node
- start the first node
- deploy system contract
  - create a account
  - create ctool json
  - deploy all system contract 
  - add the first node to NodeManager System Contract

### 3.3. four

example:

```bas
./platonectl.sh four
```

This case will do:

- create the genesis.json
- init the first node and the other three nodes
- start the first node
- deploy system contract
  - create a account
  - create ctool json
  - deploy all system contract 
  - add the first node to NodeManager System Contract
- add the other three nodes to NodeManager System Contract
- start the other three nodes 
- update the other three node type to be consensus nodes

### 3.4. start

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
        start OPTIONS
            --nodeid, -n                 start the specified node
            --bootnodes, -b              Connect to the specified bootnodes node
                                         The default is the first in the suggestObserverNodes
                                         in genesis.json
            --logsize, -s                Log block size (default: 67108864)
            --logdir, -d                 log dir (default: ../data/node_dir/logs/)
            														 The path connector '/' needs to be escaped
                                         when set: eg ".\/logs"
            --extraoptions, -e           extra platone command options when platone starts
                                         (default: --debug)
            --all, -a                    start all node
            --help, -h                   show help
```

### 3.5. stop

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
        stop OPTIONS
            --nodeid, -n                 stop the specified node
            --all, -a                    stop all node
            --help, -h                   show help
```

### 3.6. restart

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
        restart OPTIONS
            --nodeid, -n                 restart the specified node
            --all, -a                    restart all node
            --help, -h                   show help

```

### 3.7. console

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
        console OPTIONS
            --opennodeid , -n            open the specified node console
                                         set the node id here
            --closenodeid, -c            stop the specified node console
                                         set the node id here
            --closeall                   stop all node console
            --help, -h                   show help
```

### 3.8. deploysys

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
        deploysys OPTIONS
            --nodeid, -n                 the specified node id (default: 0)
            --auto                       auto=true: will use the default node password: 0
                                         to create the account and also
                                         to unlock the account (default: false)
            --help, -h                   show help

```

### 3.9. updatesys

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
        updatesys OPTIONS
            --nodeid, -n                 the specified node id
            --content, -c                update content (default: '{"type":1}')
            														 note the parameter format
            --help, -h                   show help

```

### 3.10. addnode

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
       addnode OPTIONS
           --nodeid, -n                 the specified node id. must be specified
           --desc                       the specified node desc
           --p2p_port                   the specified node p2p_port
                                        If the node specified by nodeid is local,
                                        then you do not need to specify this option.
           --rpc_port                   the specified node rpc_port
                                        If the node specified by nodeid is local,
                                        then you do not need to specify this option.
           --ip                         the specified node ip
                                        If the node specified by nodeid is local,
                                        then you do not need to specify this option.
           --pubkey                     the specified node pubkey
                                        If the node specified by nodeid is local,
                                        then you do not need to specify this option.
           --account                    the specified node account
                                        If the node specified by nodeid is local,
                                        then you do not need to specify this option.
           --help, -h                   show help
```

### 3.11. clear

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
       clear OPTIONS
           --nodeid, -n                 clear specified node data
           --all, -a                    clear all nodes data
           --help, -h                   show help

```

### 3.12. unlock

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
       unlock OPTIONS
           --nodeid, -n                 unlock specified node account
           --help, -h                   show help

```

### 3.13. get

get all nodes from NodeManager System Contract

example:

```bash
./platonectl.sh get
```

### 3.14. setupgen

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
       setupgen OPTIONS
           --nodeid, -n                 the first node id (default: 0)
           --ip                         the first node ip (default: 127.0.0.1)
           --p2p_port                   the first node p2p_port (default: 16791)
           --auto                       auto=true: Will auto create new node keys and will
                                        not compile system contracts again (default=false)
           --observerNodes, -o           set the genesis suggestObserverNodes
                                        (default is the first node enode code)
           --validatorNodes, -v         set the genesis validatorNodes
                                        (default is the first node enode code)
           --interpreter, -i            Select virtual machine interpreter in wasm, evm, all (default: wasm)
           --help, -h                   show help
```

### 3.15. status

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
       status OPTIONS                   show all node status
           --nodeid, -n                 show the specified node status info
           --all, -a                    show all  node status info
           --help, -h                   show help

```

### 3.16. createacc

```bash
    DESCRIPTION
        The deployment script for platone
    Usage:
        platonectl.sh <command> [command options] [arguments...]
    COMMANDS
       createacc OPTIONS
           --nodeid, -n                 create account for specified node
           --help, -h                   show help
```

## 4. Multi-machine deployment

Case: A, B, C, D four hosts (removing the original related file data before deployment)

* A: 172.26.10.96
* B: 172.26.10.97
* C: 172.26.10.98
* D: 172.26.10.99 

### 4.1. Production Environment Configuration Instructions

- Before each deployment, each node's machine needs to do a time synchronization.
- Log level: `-e` specifies **extra parameters**, (when `-e` is not specified) Default **Extra parameters** is `--debug`. In the production environment, you can override the log level by `-e " "` to enable the `non-debug` mode. At the same time `-e '--verbosity 2'` can be used to specify the log level.
- Log location: The production environment needs to specify the log storage path. Refer to the `start` parameter `-d` in `section 2.4` to specify the log storage location.
- Log chunking: The default chunk size is 64M. If you need to specify, you can refer to the parameter `--logsize` of `start` in `2.4 section` to specify the block size.
- Specify the synchronization node via `--bootnodes`, **It is recommended to use the suggestObserverNoes in genesis.json to connect **, which can be specified in `-e " "` (as shown in the following example).

Therefore, in a production environment:

The steps in the `4.2` section refer to the `start` startup step:

 `./platonectl.sh start -n x`

Need to be replaced with:

`./platonectl.sh start -n x -d "\/opt\/logs" -e "--bootnodes enode://8ab91d36a58e03c7d5528ea9186474cf5bfbec46d24cd59cf5eef1b63b2f4120334ca2a6af9ae495fa1931cdfe684caa74c86ad77fcfa0f044f4da30f7a83a4e@127.0.0.1:16791" ` (The specific path depends on the production plan)

### 4.2. start

in A:

```bash
cd ${WORKSPACE}/scripts
./platonectl.sh clear -a
./platonectl.sh setupgen --ip 172.26.10.96 --auto true
./platonectl.sh init -n 0 --ip 172.26.10.96 --auto true
./platonectl.sh start -n 0
./platonectl.sh deploysys --auto true
./platonectl.sh init -n 1 --ip 172.26.10.97 --auto true
./platonectl.sh init -n 2 --ip 172.26.10.98 --auto true
./platonectl.sh init -n 3 --ip 172.26.10.99 --auto true
for n in $(seq 3); do
	  ./platonectl.sh addnode -n ${n}
done
```

The multi-machine data files required at this time are fully prepared, and the data is copied to the other three hosts.

```bash
scp -r ${WORKSPACE}/../linux user@172.26.10.97:~/
scp -r ${WORKSPACE}/../linux user@172.26.10.98:~/
scp -r ${WORKSPACE}/../linux user@172.26.10.99:~/
```

in B:

```bash
cd ~/linux/scripts
./platonectl.sh start -n 1
```

in C:

```bash
cd ~/linux/scripts
./platonectl.sh start -n 2
```

in D:
```bash
cd ~/linux/scripts
./platonectl.sh start -n 3
```

### 4.3. Update node

in A:

```bash
./platonectl.sh updatesys -n 1
./platonectl.sh updatesys -n 2
./platonectl.sh updatesys -n 3
```



## 5. Backup

This function supports the node not starting, and the data in the chaindb is damaged. By passing the block data offline, the block data behind a node is filled.

### 5.1. export

Export the RLP-encoded block data in a specified range of a node to a file by using the export function.

```
./platone --datadir <chaindata path of the node to be exported> export <output file name> <export block height lower bound> <export block height upper bound>
```

### 5.2. import

Import the block data exported in 7.1 to the specified node through the import function.

```
./platone --datadir <chaindata path of the node to be imported> import <block file name>
```

## 6. Production log cleanup policy reference

We simulated the amount of logs under normal trading pressure: on a single node, it produced a log of about 300M in 24 hours.

- Assume that under the planning of the 500G data disk, according to the 70% threshold retention, except the chain DB data (recommended to retain at least 100GB), then you can retain about 27 months of data.
- However, due to the possibility of peak transaction, it is recommended to implement a cleanup strategy for the space size threshold at the same time, that is, when the total amount of logs reaches 500GB*70%-100GB=250GB, the first month of data is cleaned up.

**to sum up:**

* The log cleanup strategy for both time and space dimensions is implemented simultaneously.

## 7. Operation status check & error troubleshooting

Before delivering the chain to the business, we can verify the correctness of the chain from the following dimensions, including but not limited to the following steps:

* Chain running status check: Run the log in the chain to see if it is normal. (The normal block interval is between 1 and 3 seconds)
* System contract deployment check:
  * The deployment log of the system contract is in the `wasm_log` folder. You can monitor whether the error keyword appears in the log and check whether the contract is deployed normally.
  * Confirm that all nodes have been logged to the node management contract via the `./platonectl.sh get` command.

* Monitor the `error` or `warning` keywords of the chain running process.
  * Partially related to the instantaneous connectivity of the node, such as the error message caused by the node ping heartbeat packet can be ignored.