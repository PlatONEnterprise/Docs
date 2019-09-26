`快速安装、搭建、测试PlatONE网络`
## 安装PlatONE
PlatONE支持Linux运行环境，Linux环境下完整的安装教程请参考PlatONE[安装指南](zh-cn/basics/[Chinese-Simplified]-安装指南)。

## 搭建私有网络
如果你希望在本地测试或运行PlatONE，可以参考[安装指南](zh-cn/basics/[Chinese-Simplified]-安装指南), 或者参考以下快速搭链步骤。

```bash
# 获取PlatONE源码
git clone --recursive https://github.com/PlatONEnterprise/PlatONE-Go.git
export WORKSPACE=${PWD}/PlatONE-Go/release/linux
# 编译PlatONE
cd PlatONE-Go; make all; cd ..
# 如果编译失败. 请确保你安装了 1.3.1中全部软件. 然后重新尝试.
```

### 快速搭链
用户可以使用脚本快速的搭建一个四节点的链以供测试。
```
cd ${WORKSPACE}/scripts
./plantonectl.sh four
```

## 创建PlatONE账户
在发布交易之前，首先用户需要创建一个账户。

```
cd ${WORKSPACE}/../../cmd/SysContracts/

./script/create_accout.sh --ip 127.0.0.1 --rpc_port 6791
```

该命令会创建一个新的账户，创建时需要输入一个账户密码，方便后续解锁。
运行结果如下：

```
nodeid: 127.0.0.1
rpc_port: 6791

###########################################
####       Create an account           ####
###########################################

Input account passphrase.
passphrase: 000
--output{"jsonrpc":"2.0","id":1,"result":"0x08e7988e60ab5aa49d1f7aa9435ac91b9fcf772c"} --output
New account: 0x08e7988e60ab5aa49d1f7aa9435ac91b9fcf772c
{"jsonrpc":"2.0","id":1,"result":true}
127.0.0.1:6791

 Create config.json for contract-deploy

{
  "url":"http://127.0.0.1:6791",
  "gas":"0x0",
  "gasPrice":"0x0",
  "from":"0x08e7988e60ab5aa49d1f7aa9435ac91b9fcf772c"
}
```
## 解锁账户
长时间没用使用账户发送过交易，或者节点重启后，账户会被锁定，不能发送交易。需要使用如下命令，重新为账户解锁。

```
cd ${WORKSPACE}/../../cmd/SysContracts/
./script/unlock_accout.sh \
            --account 0x08e7988e60ab5aa49d1f7aa9435ac91b9fcf772c \
            --phrase "your phrase"
            --ip 127.0.0.1 \
            --rpc_port 6791
```
## 部署合约

```
cd ${WORKSPACE}/../../cmd/SysContracts/build

# ctool是用来部署合约及发送交易的工具
cp ${WORKSPACE}/bin/ctool .  

# 部署合约
./ctool deploy --abi appContract/appDemo/appDemo.cpp.abi.json --code appContract/appDemo/appDemo.wasm --config ../config.json
```

结果:
```
trasaction hash: 0x2f3a868d8ceb60804a830e5fc35611e3ae22ccb6ef298830acd84aba41f6dd99
contract address: 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206
```
## 调用合约

调用合约的`invokeNotify方法`

```
./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/appDemo/appDemo.cpp.abi.json   --config ../config.son  --func "invokeNotify" --param "wxbc"  

 request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xdb8800000000000000028c696e766f6b654e6f746966798477786263","txType":2}] 

 response json：{"jsonrpc":"2.0","id":1,"result":"0xeb3680c65b393952a07ec590cef7b19fc87877a9fbb70c8e16797a97ed4cfaeb"}
```

调用合约的`getName`方法：

```
./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/my_contract/my_contract.cpp.abi.json   --config ./config.json  --func getName

request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xd1880000000000000002876765744e616d65","txType":2},"latest"] 

response json：{"jsonrpc":"2.0","id":1,"result":"0x000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000047778626300000000000000000000000000000000000000000000000000000000"}
 
result: wxbc
```
## 通过控制台连接到节点
可通过以下http的方式进入PlatONE控制台。
```
cd  ${WORKSPACE}/bin

./platone attach http://127.0.0.1:6791
```
命令执行的结果如下：
```
Welcome to the PlatONE JavaScript console!

instance: PlatONEnetwork/platone/v0.2.0-stable-1b13ff73/linux-amd64/go1.12.4
coinbase: 0x938c231429f5ab34244618fe0dc7380e319b470e
at block: 1557 (Tue, 18 Jun 2019 16:29:12 CST)
 datadir: /home/gexin/PlatONE-Workspace/chain/PlatONE_linux/data/node-1
 modules: admin:1.0 eth:1.0 net:1.0 personal:1.0 rpc:1.0 web3:1.0

> 
```
## 查询交易receipt

```
> eth.getTransactionReceipt("0x2f3a868d8ceb60804a830e5fc35611e3ae22ccb6ef298830acd84aba41f6dd99")
```

结果如下：

```js
{
  blockHash: "0x3f1e326f4b122efb26e9da417daff97dbb403c714d3324e8020ddce04b88617a",
  blockNumber: 236,
  contractAddress: "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206",
  cumulativeGasUsed: 1996535,
  from: "0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87",
  gasUsed: 1996535,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x1",
  to: null,
  transactionHash: "0x2f3a868d8ceb60804a830e5fc35611e3ae22ccb6ef298830acd84aba41f6dd99",
  transactionIndex: 0
}
```

