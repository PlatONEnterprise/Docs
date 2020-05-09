
本文档第1章节, 介绍使用脚本部署PlatONE的简单案例演示；第2章节，介绍了脚本各个命令的功能以及使用方式；第3章节详细的列出脚本各个命令的命令行参数。以上3个章节讲述的是PlatONE在单机上进行脚本部署的方式，第4章节进行演示了PlatONE如何在多机上的脚本部署方式。至于第5、6和7 章节则给出了PlatONE在备份、日志清理策略以及运行状态检查的等运维维护方面的建议。

## 0. 环境依赖

* gcc 7.3+
* g++ 7.3+
* cmake 3.10+
* go 1.11.4+

## 1. 案例演示(适用于简单测试环境）

如果您是第一次启动PlatONE区块链，最好的选择是启动一个节点，然后将其他节点添加到链中。 在第1.1.部分和1.2 .我们简要介绍了如何启动链并向链中添加新节点。 此外，部署区块链的更多细节可以在第2章节中看到。

```bash
# 获取PlatONE源码
git clone --recursive https://github.com/PlatONEnterprise/PlatONE-Go.git
export WORKSPACE=${PWD}/PlatONE-Go/release/linux
# 编译PlatONE
cd PlatONE-Go; make all; cd ..
# 如果编译失败. 请确保你安装了 1.3.1中全部软件. 然后重新尝试.
```

### 1.1. 快速演示: 启动单节点区块链

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

### 1.2. 快速演示: 将其他节点添加到链中

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

## 2. 脚本功能演示

### 2.1. 环境准备

在编译之前，你需要先下载如下软件:

- cmake 3.10+
- go 1.11+
- gcc 7.3+

### 2.2. 创建genesis.json

当您启动区块链时，首先需要创建genesis.json文件。

```bash
cd ${WORKSPACE}/scripts
./platonectl.sh setupgen -n 0 --ip 127.0.0.1 --p2p_port 16791 --auto true  
```

* auto=false: 将给出提示进行选择是否编译系统合约

您也可以使用默认值。

```bash
./platonectl.sh setupgen
```

### 2.3. 初始化节点

```bash
./platonectl.sh init -n 0 --ip 127.0.0.1 --rpc_port 6791 --p2p_port 16791 \
												--ws_port 26791 --auto "true"
```

您也可以使用默认值。

```bash
./platonectl.sh init
```

### 2.4. 启动节点

```bash
./platonectl.sh start -n 0
```

在启动节点时，也可以指定日志文件夹的路径，日志文件分块的大小，指定platone启动时额外的命令行参数等. (注意: 路径连接符'/' 需要进行转义，参数option的值，必须加上引号)

```bash
./platonectl.sh start -n 0 -d '.\/logs\/platone' -s 65535 -e '--verbosity 3 --debug'
```

日志文件夹中包含wasm执行的日志与platone运行的日志. 随时间推移，日志文件会越积越多，建议进行挂载，或者进行定期删除等操作。

### 2.5. 部署系统合约

```bash
./platonectl.sh deploysys -n 0 --auto true 
```

### 2.6. 添加节点

您可能希望向PlatONE网络添加一些节点，您必须首先初始化此新节点。

- 初始化新节点(node 1):

```bash
./platonectl.sh init -n 1 --ip 127.0.0.1 --rpc_port 6792 --p2p_port 16792 \
												--ws_port 26792 --auto "true"
```

- 增加新节点:

```bash
./platonectl.sh addnode -n 1
```

- 您也可以添加远程节点:

```bash
./platonectl.sh addnode -n 10 --p2p_port 3333 --rpc_port 4444 --ip xxx.xxx.xxx.xxx --pubkey xxxxx
```

### 2.7. 更新节点

将节点更新为共识节点。

```bash
./platonectl.sh updatesys -n 1 
```

您还可以更新节点状态：

```bash
./platonectl.sh updatesys -n 1 -c '{"status":2}'
```

### 2.8. 创建账户

为节点创建帐户:

```bash
./platonectl.sh createacc -n 1
```

### 2.9. 解锁帐户

解锁节点帐户。 在解锁节点帐户之前，请确保已启动节点。

```bash
./platonectl.sh unlock -n 1
```

### 2.10. 连接到console

您可以打开一个交互式JavaScript环境 :

```bash
./platonectl.sh console -n 0
```

### 2.11. 获取所有节点

您可以从系统合约中获取所有节点

```bash
./platonectl.sh get
```

### 2.12. 查看节点信息

以下命令可用于获取节点状态和信息:

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

### 2.13. 停止和清除

您可以停止节点并清除节点数据：

```bash
./platonectl.sh stop -n 0 
./platonectl.sh clear -n 0
```

它还可以简化如下:(清除包括暂停功能）:

```bash
./platonectl.sh clear -n 0
```

### 2.14. 一键启动（适用于简单测试环境）

本节将介绍如何快速启动一个节点或在同一台机器中启动四个节点。

**一键启动单节点**

快速启动单节点区块链:

```bash
./platonectl.sh one
```

**一键启动四节点**

在一台机器上快速启动四节点区块链:

```bash
./platonectl.sh four
```

## 3. 命令详情

本节详细介绍了命令。

```console
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令：
        init                             初始化节点。 请先设置genesis
        one                              完全启动一个节点
                                         默认帐户密码：0
        four                             完全启动四个节点
                                         默认帐户密码：0
        start                            尝试启动指定的节点
        stop                             尝试停止指定的节点
        restart                          尝试重新启动指定的节点
        console                          启动交互式JavaScript环境
        deploysys                        部署系统合约
        updatesys                        正常节点更新到共识节点
        addnode                          将正常节点添加到系统合同中
        clear                            清除所有节点数据
        unlock                           解锁节点帐户
        get                              显示系统合同中的所有节点
        setupgen                         创建genesis.json并编译系统契约
        status                           显示节点状态
        createacc                        创建帐户
```

### 3.1. init

例子: 

```bash
./platonectl.sh init -h
```

### 3.2. one

例子:

```bash
./platonectl.sh one 
```

这种情况下:

- 创建genesis.json

- 初始化第一个节点
- 启动第一个节点
- 部署系统合约
  - 创建用户
  - 创建ctool json
  - 部署所有系统合约
  - 将第一个节点添加到NodeManager System Contract

### 3.3. four

例子:

```bash
./platonectl.sh four
```

这种情况下:

- 创建 genesis.json
- 初始化第一个节点和其他三个
- 开启第一个节点
- 部署系统合约
  - 创建用户
  - 创建ctool json
  - 部署所有的系统合约 
  - 将第一个节点添加到 NodeManager System Contract
- 添加其他三个节点到NodeManager System Contract
- 启动其他三个节点 
- 更新其他三个节点类型为共识节点

### 3.4. start

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        start OPTIONS
            --nodeid，-n                 启动指定的节点
            --bootnodes，-b              连接到指定的bootnodes节点
                                         默认值是suggestObserverNodes中的第一个enode在genesis.json           
            --logsize，-s                日志块大小（默认值：67108864）
            --logdir，-d                 log dir (默认值位置：../data/node_dir/logs/)
            														 设置时路径连接符'/'需要进行转义: 如 ".\/logs"
            --extraoptions，-e           platone命令启动时，额外需要设置的命令行参数.
            														 (默认值: --debug)
            --all，-a                    启动所有节点
            --help，-h                   显示帮助
```

### 3.5. stop

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        stop OPTIONS
            --nodeid，-n                 停止指定的节点
            --all，-a                    停止所有节点
            --help，-h                   显示帮助
```

### 3.6. restart

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        restart OPTIONS
            --nodeid，-n                 重新启动指定的节点
            --all，-a                    重启所有节点
            --help，-h                   显示帮助

```

### 3.7. console

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        console OPTIONS
            --opennodeid ，-n             打开指定的节点控制台
                                          在这里设置节点ID
            --closenodeid，-c             停止指定的节点控制台
                                          在这里设置节点ID
            --closeall                    停止所有节点控制台
            --help，-h                    显示帮助
```

### 3.8. deploysys

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        deploysys OPTIONS
            --nodeid，-n                 指定的节点标识（默认值：0）
            --auto                       auto=true: 将使用默认节点密码：0
                                         创建帐户，并解锁帐户（默认值：false）
            --help，-h                   显示帮助

```

### 3.9. updatesys

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        updatesys OPTIONS
            --nodeid，-n                 指定的节点ID
            --content，-c                更新内容 (默认值：'{“type”：1}'）
            														 注意参数格式
            --help，-h                   显示帮助

```

### 3.10. addnode

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        addnode OPTIONS
           --nodeid，-n                 指定的节点ID。必须指定
           --desc                       指定的节点desc
           --p2p_port                   指定的节点p2p_port
                                        如果nodeid指定的节点是本地的，
                                        那么你不需要指定这个选项。
           --rpc_port                   指定的节点rpc_port
                                        如果nodeid指定的节点是本地的，
                                        那么你不需要指定这个选项。
           --ip                         指定的节点ip
                                        如果nodeid指定的节点是本地的，
                                        那么你不需要指定这个选项。
           --pubkey                     指定的节点pubkey
                                        如果nodeid指定的节点是本地的，
                                        那么你不需要指定这个选项。
           --account                    指定的节点帐户
                                        如果nodeid指定的节点是本地的，
                                        那么你不需要指定这个选项。
           --help，-h                   显示帮助
```

### 3.11. clear

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        clear OPTIONS
           --nodeid，-n                 清除指定的节点数据
           --all，-a                    清除所有节点数据
           --help，-h                   显示帮助

```

### 3.12. unlock

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        unlock OPTIONS
           --nodeid，-n                 解锁指定的节点帐户
           --help，-h                   显示帮助

```

### 3.13. get

从NodeManager系统合同中获取所有节点

例子:

```bash
./platonectl.sh get
```

### 3.14. setupgen

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        setupgen OPTIONS
           --nodeid，-n                 第一个节点id（默认值：0）
           --ip                         第一个节点ip（默认值：127.0.0.1）
           --p2p_port                   第一个节点p2p_port（默认值：16791）
           --auto                       auto=true: 将自动创建新的节点密钥并将自动创建
                                        不再编译系统合约（默认= false）
           --observerNodes，-o           设置genesis suggestObserverNodes
                                        （默认值是第一个节点的enode代码）
           --validatorNodes，-v         设置genesis validatorNodes
                                        （默认值是第一个节点的enode代码）
           --interpreter，-i            选择虚拟机解释器：wasm，evm，all (default: wasm)
           --help，-h                   显示帮助
```

### 3.15. status

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        status OPTIONS                   显示所有节点状态
           --nodeid，-n                 显示指定的节点状态信息
           --all，-a                    显示所有节点状态信息
           --help，-h                   显示帮助

```

### 3.16. createacc

```bash
    描述
        PlatONE脚本部署
    用法:
        platonectl.sh <command> [command options] [arguments...]
    命令
        createacc OPTIONS
           --nodeid，-n                 为指定节点创建帐户
           --help，-h                   显示帮助
```

## 4. 多机部署（适用于生产环境/多机测试环境）

案例:  A，B，C，D四台主机 (部署前先删除原有的相关文件数据)

* A: 172.26.10.96
* B: 172.26.10.97
* C: 172.26.10.98
* D: 172.26.10.99 

### 4.1. 生产环境配置说明

- 在部署前，每个节点的机器需要做一次时间同步。
- 日志等级：`-e` 指定了**额外参数**，（不指定` -e`的情况下）默认**额外参数**是 `--debug`。在生产环境中可以通过 `-e " "` 覆盖日志等级，即可开启`非debug`模式。同时`-e '--verbosity 2'`可以用来指定日志等级。
- 日志位置：生产环境需要指定日志存放路径，参考`2.4节`中 `start`的参数`-d`指定日志存放位置
- 日志分块：默认分块大小为64M。如需要指定 可参考`2.4节`中 `start`的参数`--logsize`指定分块大小。
- 通过`--bootnodes`指定同步节点，**推荐使用genesis.json中的suggestObserverNoes来连接**，可在`-e " "`中指定（如下例子所示）。

因此，在生产环境中：

`4.2`节中涉及到`start`启动步骤的操作步骤：

 `./platonectl.sh start -n x`

需要替换成：

`./platonectl.sh start -n x -d "\/opt\/logs"  -e "--bootnodes enode://8ab91d36a58e03c7d5528ea9186474cf5bfbec46d24cd59cf5eef1b63b2f4120334ca2a6af9ae495fa1931cdfe684caa74c86ad77fcfa0f044f4da30f7a83a4e@127.0.0.1:16791" `  （具体路径需要视生产的规划）

### 4.2. 启动

A中:

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

此时所需要的多机数据文件准备齐全，拷贝数据到其他三台主机

```bash
scp -r ${WORKSPACE}/../linux user@172.26.10.97:~/
scp -r ${WORKSPACE}/../linux user@172.26.10.98:~/
scp -r ${WORKSPACE}/../linux user@172.26.10.99:~/
```

B中:

```bash
cd ~/linux/scripts
./platonectl.sh start -n 1
```

C中:

```bash
cd ~/linux/scripts
./platonectl.sh start -n 2
```

D中:

```bash
cd ~/linux/scripts
./platonectl.sh start -n 3
```

### 4.3. 更新节点为共识节点

A中:

```bash
./platonectl.sh updatesys -n 1
./platonectl.sh updatesys -n 2
./platonectl.sh updatesys -n 3
```

## 5. 备份

该功能支持节点未启动，以及chaindb中数据损坏的场景下，通过线下传递区块数据的方式，将某节点落后的区块数据补齐
### 5.1. export
通过export功能，将某节点指定范围内的经过RLP编码后的区块数据导出到某个文件中
```
./platone --datadir <待导出节点的chaindata路径> export <输出文件名> <导出区块高度下界> <导出区块高度上界>
```


### 5.2. import

通过import功能，将7.1中导出的区块数据导入指定节点
```
./platone --datadir <待导入节点的chaindata路径> import <区块文件名>
```

## 6. 生产日志清理策略参考

我们模拟了正常交易压力下的日志量：单节点上，24小时产出约为300M大小的日志。

* 假设在500G数据盘的规划下，按照70%的阈值保留，去除链DB数据（建议保留至少100GB），那么可以保留约27个月的数据。

* 但由于交易峰值出现的可能性，建议同时实施空间大小阈值的清理策略，即当日志总量达到500GB*70%-100GB =250GB 时，实施对最早的一个月数据的清理。

**总结**

* 时间维度和空间维度的日志清理策略同时实施。

##  7. 运行状态检查&错误排查

在将链交付给业务前，我们可以从以下维度验证链的运行正确性，包括但不限于以下步骤：

* 链运行状态检查：链运行日志，观察是否正常出块。（正常出块间隔在1～3秒之间）
* 系统合约部署情况检查：
  * 系统合约的部署日志在 wasm_log文件夹中，可以监控日志中是否出现了 error 关键词，排查合约是否正常部署。
  * 通过`./platonectl.sh get` 命令，确认所有节点已经被记录到了节点管理合约。

* 监控链运行过程的`error`或者`warning`关键词。
  * 部分和节点瞬时联通性相关的，如节点互ping心跳包导致的报错信息可忽略

