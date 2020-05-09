本文档第1章节介绍PlatONE运行所需要具备的环境，建议用户预先安装好此章节中的软件，然后再进行后续章节的操作；第2章节详细说明了PlatONE如何编译以及部署，并给出了具体的操作步骤；第3章节演示了如何初始化和启动PlatONE；第4章节详解介绍了系统合约的部署以及调用方式。

## 1. 准备工作

PlatONE git的仓库地址为:

命令行输入以下命令，或者设置为环境变量。

```bash
REPO_ADDR_HTTP="https://github.com/PlatONEnterprise/PlatONE-Go.git"
REPO_ADDR_GIT="git@github.com:PlatONEnterprise/PlatONE-Go.git"
```

### 1.1. 环境

* gcc 7.3+
* g++ 7.3+
* cmake 3.10+
* go 1.11.4+

## 2. PlatONE编译和部署

下载源码：

git clone时可以使用两种方法，一种是https协议，一种是git协议。

第一种方式：
使用https协议，需要输入主机当前账户的用户名和密码

``` bash
git clone --recursive  ${REPO_ADDR_HTTP}
```
第二种方式：使用git协议
``` bash
git clone --recursive ${REPO_ADDR_GIT}
```

编译：

``` console
cd PlatONE-Go; make all
```

## 3. PlatONE节点初始化和运行

### 3.1. PlatONE节点初始化
#### 3.1.1. 生成 account 和 key pair
1) 配置环境变量，进入PlatONE-Go/build/bin

``` console 
export PATH=${PATH}:${PWD}
```
2) 生成新的**用户账户**，需要用户设置密码用于解锁用户账户，在示例中密码设为“0”。

```console
$ ./platone --datadir ./data account new

INFO [01-09|17:25:14.269] Maximum peer count                       ETH=50 LES=0 total=50
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase: 
Repeat passphrase: 
Address: {60208c048e7eb8e38b0fac40406b819ce95aa7af}
```
3) 查看账户

```console
$ ll data/keystore/

-rw------- 1 wxuser wxuser 491 Jan  9 17:25 UTC--2019-01-09T09-25-28.487164507Z--60208c048e7eb8e38b0fac40406b819ce95aa7af
```
4) 生成**节点**密钥对，需要进入目录PlatONE-Go/build/bin

```console
$ ./ethkey genkeypair

Address   :  0xC71433b47f1b0053f935AEf64758153B24cE7445
PrivateKey:  b428720a89d003a1b393c642e6e32713dd6a6f82fe4098b9e3a90eb38e23b6bb
PublicKey :  68bb049008c7226de3188b6376127354507e1b1e553a2a8b988bb99b33c4d995e426596fc70ce12f7744100bc69c5f0bce748bc298bf8f0d0de1f5929850b5f4
```

输出说明：

* Address:  节点地址。
* PrivateKey: 节点私钥。
* PublicKey: 节点公钥。

5) 将节点私钥存储在 ./data/platone/nodekey中，私钥是上一步生成的PrivateKey。

```console
$ mkdir -p ./data/platone

$ echo "b428720a89d003a1b393c642e6e32713dd6a6f82fe4098b9e3a90eb38e23b6bb" > ./data/platone/nodekey

$ cat ./data/platone/nodekey

$ sudo updatedb

$ locate nodekey

/home/wxuser/work/golang/src/github.com/PlatONEnetwork/PlatONE-Go/build/bin/data/platone/nodekey
```
#### 3.1.2. 生成系统管理合约cnsProxy字节码文件
1)进入PlatONE-Go/cmd/SysContracts目录，执行脚本生成makefile文件。

```
./script/build_system_contracts.sh
```
2)进入PlatONE-Go/cmd/SysContracts/build/systemContract/cnsProxy目录，执行ctool，获取字节码。

```
ctool codegen --abi cnsProxy.cpp.abi.json --code cnsProxy.wasm
```
该字节码将放入后续 3.1.3 的genesis.json配置文件当中。

#### 3.1.3. 配置初始化文件

生成genesis.json文件：

- validatorNodes，suggestObserverNodes中enode格式为‘enode://publicKey@ip:p2p_port’，需把在3.1.1中的 `4)`小节中生成的节点publicKey替换此enode中publicKey。ip和p2p_port可以根据情况自定义。
- alloc：为用户账户地址分配金额。用户账户地址在3.1.1章的第`2)`小结生成
- 0x0000000000000000000000000000000000000011为系统管理合约，此为固定地址。
- code：为上节中所获取的cnsProxy合约的字节码。

```console
$ vi genesis.json

{
    "config": {
    "chainId": 300,
    "homesteadBlock": 1,
    "eip150Block": 2,
    "eip150Hash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "eip155Block": 3,
    "eip158Block": 3,
    "byzantiumBlock": 4,
    "istanbul": {
            "timeout": 2000,
          "period": 1,
          "policy": 0,
          "epoch": 1000000,
          "initialNodes": [],
        "validatorNodes": ["enode://68bb049008c7226de3188b6376127354507e1b1e553a2a8b988bb99b33c4d995e426596fc70ce12f7744100bc69c5f0bce748bc298bf8f0d0de1f5929850b5f4@127.0.0.1:16789"],
        "suggestObserverNodes": ["enode://68bb049008c7226de3188b6376127354507e1b1e553a2a8b988bb99b33c4d995e426596fc70ce12f7744100bc69c5f0bce748bc298bf8f0d0de1f5929850b5f4@127.0.0.1:16789"]
    }
  },
  "nonce": "0x0",
  "timestamp": "0x5c074288",
  "extraData": "0x00000000000000000000000000000000000000000000000000000000000000007a9ff113afc63a33d11de571a679f914983a085d1e08972dcb449a02319c1661b931b1962bce02dfc6583885512702952b57bba0e307d4ad66668c5fc48a45dfeed85a7e41f0bdee047063066eae02910000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "gasLimit": "0x47b77760",
  "difficulty": "0x40000",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x60208c048e7eb8e38b0fac40406b819ce95aa7af",
  "alloc": {
    "0x60208c048e7eb8e38b0fac40406b819ce95aa7af": {
      "balance": "99999999900000000000"
    },
    "0x0000000000000000000000000000000000000011": {
      "balance": "99900000000000000000",
      "code": "cnsProxy字节码"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

#### 3.1.4. 初始化platone
```console
$ platone --datadir ./data init genesis.json

INFO [01-09|17:31:58.832] Maximum peer count                       ETH=50 LES=0 total=50
INFO [01-09|17:31:58.833] Allocated cache and file handles         database=/home/wxuser/manual-Platone/build/bin/data/platone/chaindata cache=16 handles=16
INFO [01-09|17:31:58.839] Writing custom genesis block 
INFO [01-09|17:31:58.840] Persisted trie from memory database      nodes=1 size=150.00B time=34.546µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [01-09|17:31:58.840] Successfully wrote genesis state         database=chaindata                                                  hash=4fe06b…382a26
INFO [01-09|17:31:58.840] Allocated cache and file handles         database=/home/wxuser/manual-Platone/build/bin/data/platone/lightchaindata cache=16 handles=16
INFO [01-09|17:31:58.848] Writing custom genesis block 
INFO [01-09|17:31:58.848] Persisted trie from memory database      nodes=1 size=150.00B time=238.177µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [01-09|17:31:58.848] Successfully wrote genesis state         database=lightchaindata                                                  hash=4fe06b…382a26
```

```console
$ ll -R data/

data/:
total 0
drwx------ 2 wxuser wxuser 91 Jan  9 17:25 keystore
drwxr-xr-x 4 wxuser wxuser 45 Jan  9 17:31 platone

data/keystore:
total 4
-rw------- 1 wxuser wxuser 491 Jan  9 17:25 UTC--2019-01-09T09-25-28.487164507Z--60208c048e7eb8e38b0fac40406b819ce95aa7af

data/platone:
total 0
drwxr-xr-x 2 wxuser wxuser 85 Jan  9 17:31 chaindata
drwxr-xr-x 2 wxuser wxuser 85 Jan  9 17:31 lightchaindata

data/platone/chaindata:
total 16
-rw-r--r-- 1 wxuser wxuser 1802 Jan  9 17:31 000001.log
-rw-r--r-- 1 wxuser wxuser   16 Jan  9 17:31 CURRENT
-rw-r--r-- 1 wxuser wxuser    0 Jan  9 17:31 LOCK
-rw-r--r-- 1 wxuser wxuser  358 Jan  9 17:31 LOG
-rw-r--r-- 1 wxuser wxuser   54 Jan  9 17:31 MANIFEST-000000

data/platone/lightchaindata:
total 16
-rw-r--r-- 1 wxuser wxuser 1802 Jan  9 17:31 000001.log
-rw-r--r-- 1 wxuser wxuser   16 Jan  9 17:31 CURRENT
-rw-r--r-- 1 wxuser wxuser    0 Jan  9 17:31 LOCK
-rw-r--r-- 1 wxuser wxuser  358 Jan  9 17:31 LOG
-rw-r--r-- 1 wxuser wxuser   54 Jan  9 17:31 MANIFEST-000000
```

### 3.2. 启动 platone 节点
```console
$ platone --identity "platone" --datadir ./data --port 16789 --rpcaddr 0.0.0.0 --rpcport 6789 --rpcapi "db,eth,net,web3,admin,personal" --rpc --nodiscover --nodekey "./data/platone/nodekey" --verbosity 4 --wasmlog ./wasm.log --bootnodes "enode://68bb049008c7226de3188b6376127354507e1b1e553a2a8b988bb99b33c4d995e426596fc70ce12f7744100bc69c5f0bce748bc298bf8f0d0de1f5929850b5f4@127.0.0.1:16789"

(注：--verbosity 4 会将wasm log打出来，--wasmlog 指定将log输出到哪个文件，--bootnodes需要指定genesis.json中suggestObserverNodes字段中的一个或者多个enode节点)

INFO [01-09|17:42:01.165] Maximum peer count                       ETH=50 LES=0 total=50
INFO [01-09|17:42:01.166] Starting peer-to-peer node               instance=Geth/node1/v1.8.16-stable-7ee6fe39/linux-amd64/go1.11.4
INFO [01-09|17:42:01.166] Allocated cache and file handles         database=/home/wxuser/manual-Platone/build/bin/data/platone/chaindata cache=768 handles=512
INFO [01-09|17:42:01.183] Initialised chain configuration          config="{ChainID: 300 Homestead: 1 DAO: <nil> DAOSupport: false EIP150: 2 EIP155: 3 EIP158: 3 Byzantium: 4 Constantinople: <nil> Engine: &{0 0 0 0 0 [{127.0.0.1 16789 16789 68bb049008c7226de3188b6376127354507e1b1e553a2a8b988bb99b33c4d995e426596fc70ce12f7744100bc69c5f0bce748bc298bf8f0d0de1f5929850b5f4 [149 178 250 27 246 47 49 86 100 108 50 3 199 20 51 180 127 27 0 83 249 53 174 246 71 88 21 59 36 206 116 69] {0 0 <nil>}}] 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000 <nil>}}"
INFO [01-09|17:42:01.183] Initialising Ethereum protocol           versions="[63 62]" network=300
INFO [01-09|17:42:01.184] Loaded most recent local header          number=0 hash=4fe06b…382a26 age=1mo5d6h
INFO [01-09|17:42:01.184] Loaded most recent local full block      number=0 hash=4fe06b…382a26 age=1mo5d6h
INFO [01-09|17:42:01.184] Loaded most recent local fast block      number=0 hash=4fe06b…382a26 age=1mo5d6h
INFO [01-09|17:42:01.184] Read the StateDB instance from the cache map sealHash=bbbae7…30dbfb
INFO [01-09|17:42:01.184] Loaded local transaction journal         transactions=0 dropped=0
INFO [01-09|17:42:01.185] Regenerated local transaction journal    transactions=0 accounts=0
INFO [01-09|17:42:01.185] Loaded local mpc transaction journal     mpc transactions=0 dropped=0
INFO [01-09|17:42:01.185] Init mpc processor success               osType=linux icepath= httpEndpoint=http://127.0.0.1:6789
INFO [01-09|17:42:01.185] commitDuration                           commitDuration=950.000
INFO [01-09|17:42:01.185] Set the block time at the end of the last round of consensus startTimeOfEpoch=1543979656
INFO [01-09|17:42:01.185] Starting P2P networking 
INFO [01-09|17:42:03.298] UDP listener up                          self=enode://aa18a88c1463c1f1026c6cb0b781027d898d19ed9c11b10ad7a3a9ee2d0c09ab607d9b24bc4580bd816c0194215461cd88bf65955e0d87cf69e0157d464c582b@[::]:16789
INFO [01-09|17:42:03.299] Transaction pool price threshold updated price=1000000000
INFO [01-09|17:42:03.300] IPC endpoint opened                      url=/home/wxuser/manual-Platone/build/bin/data/platone.ipc
INFO [01-09|17:42:03.300] RLPx listener up                         self=enode://aa18a88c1463c1f1026c6cb0b781027d898d19ed9c11b10ad7a3a9ee2d0c09ab607d9b24bc4580bd816c0194215461cd88bf65955e0d87cf69e0157d464c582b@[::]:16789
INFO [01-09|17:42:03.300] HTTP endpoint opened                     url=http://0.0.0.0:6789                                  cors= vhosts=localhost
INFO [01-09|17:42:03.300] Transaction pool price threshold updated price=1000000000
```

#### 3.2.1. platone 与log相关的启动参数

启动platone时，指定`--moduleLogParams` 参数可以把platone的log分块写入文件。

```bash
--moduleLogParams '{"platone_log": ["/"]，"__dir__": ["../../logs"]，"__size__": ["67108864"]}'
```

参数说明: 

* `platone_log`: 指定输出platone中哪个模块的日志。  如 `"platone_log": ["/consensus"，"/p2p"]`，则只输出consensus模块和p2p模块中打印的日志。
  * `"platone_log": ["/"]` 则表示输出所有模块的日志。
* `__dir__`: 指定的log输出的目录位置。
* `__size__`:  指定log写入文件的分块大小。

随时间推移，日志文件会越积越多，建议进行挂载，或者进行定期删除等操作。

更多的platone启动参数，可以执行以下命令，进行查看。

```bash
platone -h 
```


### 3.3. 重新初始化platone节点
确保platone进程已经被杀死，再删除data目录。（如需进行后续章节步骤，此过程需先省略）
```console
$ rm -rf data/platone
```
然后可以再重新初始化。

## 4. 系统合约部署

### 4.1 生成ctool.json

进入`PlatONE-Go/cmd/SysContracts/build/systemContract`目录，确保此时platone已启动。 使用vi创建ctool.json文件，写下如下内容。根据此时启动的节点的情况，替换如下模板中的NODE-IP、RPC-PORT、DEFAULT-ACCOUNT。

```json
vi ctool.json

{
  "url":"http://NODE-IP:RPC-PORT"，
  "gas":"0x0"，
  "gasPrice":"0x0"，
  "from":"0xDEFAULT-ACCOUNT"
}
```

* NODE-IP: 节点启动时设置的ip选项。
* RPC-PORT：节点启动是设置的rpc_port 端口。
* DEFAULT-ACCOUNT：在3.1.1第2小节创建的用户账号。

### 4.2 部署系统合约
部署系统合约前需要unlock部署合约的账户地址，首先进入到console，解锁用户账户。
```bash
$ platone attach http://NODE-IP:RPC-PORT
Welcome to the PlatONE JavaScript console!

instance: PlatONEnetwork/platone/v0.2.0-stable-56ea60ae/linux-amd64/go1.11.4
coinbase: 0x0fbd63b374002cb15aca95202fe10b63bda3fdcb
at block: 4012 (Tue，27 Aug 2019 10:54:40 CST)
 datadir: /home/wxuser/wywforfun/PlatONE-Go/build/bin/data
 modules: admin:1.0 eth:1.0 net:1.0 personal:1.0 rpc:1.0 web3:1.0

> 
```
然后解锁用户账户，需输入账号对应的密码。
```bash
>personal.unlockAccount("DEFAULT-ACCOUNT")
Unlock account DEFAULT-ACCOUNT
Passphrase: 
true

```
然后可以退出console进行合约部署
* NODE-IP，RPC-PORT，DEFAULT-ACCOUNT 的值，需要和4.1章中ctool.json中设置的值一致。

进入`PlatONE-Go/cmd/SysContracts/build/systemContract`目录

```bash
# 部署cnsManager系统合约
ctool deploy --config ctool.json --code cnsManager/cnsManager.wasm --abi cnsManager/cnsManager.cpp.abi.json
# 部署paramManager系统合约
ctool deploy --config ctool.json --code paramManager/paramManager.wasm --abi paramManager/paramManager.cpp.abi.json
# 部署userManager系统合约
ctool deploy --config ctool.json --code userManager/userManager.wasm --abi userManager/userManager.cpp.abi.json
# 部署userRegister系统合约
ctool deploy --config ctool.json --code userRegister/userRegister.wasm --abi userRegister/userRegister.cpp.abi.json
# 部署roleManager系统合约
ctool deploy --config ctool.json --code roleManager/roleManager.wasm --abi roleManager/roleManager.cpp.abi.json
# 部署roleRegister系统合约
ctool deploy --config ctool.json --code roleRegister/roleRegister.wasm --abi roleRegister/roleRegister.cpp.abi.json
# 部署nodeManager系统合约
ctool deploy --config ctool.json --code nodeManager/nodeManager.wasm --abi nodeManager/nodeManager.cpp.abi.json
# 部署nodeRegister系统合约
ctool deploy --config ctool.json --code nodeRegister/nodeRegister.wasm --abi nodeRegister/nodeRegister.cpp.abi.json
```

**系统合约说明**

* cnsManager：维护合约名字到合约信息（合约地址和版本等等信息）的映射。
* paramManager：可以动态跟新PlatONE运行过程中的一些系统参数。
* userManager：管理PlatONE中的用户以及用户信息。 
* userRegister：提供申请注册成为平台用户的功能。
* roleManager：管理平台的角色和角色所具备的权限。
* roleRegister：提供申请注册获取平台角色的功能。
* nodeManager：管理平台的节点以及节点信息。
* nodeRegister：提供申请注册成为平台节点的功能（非必要）。

### 4.3 系统合约调用

本节将以添加节点、查看所有节点以及更新节点信息为案例，演示系统合约的调用方式。 

首先进入`PlatONE-Go/cmd/SysContracts/build/systemContract`目录

**添加节点**

```bash
ctool cnsInvoke --cns "__sys_NodeManager"   --config ctool.json --abi nodeManager/nodeManager.cpp.abi.json --func add --param '{"name":"test","type":0,"publicKey":"68bb049008c7226de3188b6376127354507e1b1e553a2a8b988bb99b33c4d995e426596fc70ce12f7744100bc69c5f0bce748bc298bf8f0d0de1f5929850b5f4","desc":"","externalIP":"127.0.0.1","internalIP":"127.0.0.1","rpcPort":6792,"p2pPort":16792,"owner":"","status":1}'
```

其中选项`--param`部分参数说明：

* name: 节点名字。节点管理合约中，节点名字必须唯一。
* type: 节点类型。type为0表示观察者节点，type为1表示共识节点。添加节点时，type得设置为0，后续在节点稳定运行之后，可以更新节点信息设置type为1，让此节点进行参与共识。
* publicKey: 节点公钥。节点公钥可在3.1.1章节的第4小节获取。
* status: 节点状态。当status设置为1时，表示此节点为正常状态， 可以与其他节点进行连接；当status设置为2时，表示此节点为删除状态，此时将断开与其他节点的连接。

**查看所有节点**

当添加了节点之后，运行如下命令，可以查看到此时系统合约中所有添加的节点的所有信息：

```bash
ctool cnsInvoke --cns "__sys_NodeManager"   --config ctool.json --abi nodeManager/nodeManager.cpp.abi.json --func "getAllNodes"
```

**更新节点信息**

当需要跟新节点信息，需调用节点管理合约的更新方法。如把普通节点转换为共识节点，命令如下：

```bash
ctool cnsInvoke --cns "__sys_NodeManager"   --config ctool.json --abi nodeManager/nodeManager.cpp.abi.json --func "update" --param "test" --param '{"type":1}'
```

此命令中，第一个param选项指定的是需要跟新的节点的名字， 第二个param选项指定的是需要更新的节点的信息。