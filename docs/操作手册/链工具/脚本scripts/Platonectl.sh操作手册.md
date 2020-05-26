# Platonectl.sh

## 1. 命令详情

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

### 1.1 init

例子:

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

例子: platonectl.sh init -n 1
                            --ip 127.0.0.1
                            --rpc_port 6790
                            --p2p_port 16790
                            --ws_port 26790
                            --auto "true"
    or:
        platonectl.sh init
```

### 1.2 one

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

### 1.3 four

例子:

```bas
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

### 1.4 start

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    start OPTIONS
        --nodeid, -n                 启动指定的节点
        --bootnodes, -b              连接到指定的bootnodes节点
                                     默认值是observeNodes中的第一个enode在genesis.json
        --logsize, -s                日志块大小（默认值：67108864）
        --logdir, -d                 log dir (默认值位置：../data/node_dir/logs/)
                                     设置时路径连接符'/'需要进行转义: 如 ".\/logs"
        --extraoptions, -e           platone命令启动时, 额外需要设置的命令行参数.
                                     (默认值: --debug)
        --all, -a                    启动所有节点
        --help, -h                   显示帮助
```

### 1.5 stop

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    stop OPTIONS
        --nodeid, -n                 停止指定的节点
        --all, -a                    停止所有节点
        --help, -h                   显示帮助
```

### 1.6 restart

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    restart OPTIONS
        --nodeid, -n                 重新启动指定的节点
        --all, -a                    重启所有节点
        --help, -h                   显示帮助
```

### 1.7 console

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    console OPTIONS
        --opennodeid , -n             打开指定的节点控制台
                                      在这里设置节点ID
        --closenodeid, -c             停止指定的节点控制台
                                      在这里设置节点ID
        --closeall                    停止所有节点控制台
        --help, -h                    显示帮助
```

### 1.8 deploysys

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    deploysys OPTIONS
        --nodeid, -n                 指定的节点标识（默认值：0）
        --auto                       auto=true: 将使用默认节点密码：0
                                     创建帐户，并解锁帐户（默认值：false）
        --help, -h                   显示帮助
```

### 1.9 updatesys

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    updatesys OPTIONS
        --nodeid, -n                 指定的节点ID
        --content, -c                更新内容 (默认值：'{“type”：1}'）
                                     注意参数格式
        --help, -h                   显示帮助
```

### 1.10 addnode

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    addnode OPTIONS
        --nodeid, -n                 指定的节点ID。必须指定
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
        --help, -h                   显示帮助
```

### 1.11 clear

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    clear OPTIONS
        --nodeid, -n                 清除指定的节点数据
        --all, -a                    清除所有节点数据
        --help, -h                   显示帮助
```

### 1.12 unlock

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    unlock OPTIONS
        --nodeid, -n                 解锁指定的节点帐户
        --help, -h                   显示帮助
```

### 1.13 get

从NodeManager系统合同中获取所有节点

例子:

```bash
./platonectl.sh get
```

### 1.14 setupgen

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    setupgen OPTIONS
        --nodeid, -n                 第一个节点id（默认值：0）
        --ip                         第一个节点ip（默认值：127.0.0.1）
        --p2p_port                   第一个节点p2p_port（默认值：16791）
        --auto                       auto=true: 将自动创建新的节点密钥并将自动创建
                                     不再编译系统合约（默认= false）
        --observeNodes, -o           设置genesis observeNodes
                                    （默认值是第一个节点的enode代码）
        --validatorNodes, -v         设置genesis validatorNodes
                                    （默认值是第一个节点的enode代码）
        --interpreter, -i            选择虚拟机解释器：wasm, evm, all (default: wasm)
        --help, -h                   显示帮助
```

### 1.15 status

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    status OPTIONS                   显示所有节点状态
        --nodeid, -n                 显示指定的节点状态信息
        --all, -a                    显示所有节点状态信息
        --help, -h                   显示帮助
```

### 1.16 createacc

```bash
描述
    PlatONE脚本部署
用法:
    platonectl.sh <command> [command options] [arguments...]
命令
    createacc OPTIONS
        --nodeid, -n                 为指定节点创建帐户
        --help, -h                   显示帮助
```
