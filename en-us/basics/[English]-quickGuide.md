`Quickstart: Installing and running PlatONE nodes`

## Building a private network
If you want to test or run PlatONE locally, see [Installation Guide](en-us/basics/[English]-installation), or the instructions below.

```bash
# environment
# g++ 7.3+
# cmake 3.10+
# go 1.11.4+

# get PlatONE source code 
git clone --recursive https://github.com/PlatONEnterprise/PlatONE-Go.git
export WORKSPACE=${PWD}/PlatONE-Go/release/linux
# compile the PlatONE
cd PlatONE-Go; make all; cd ..
```

### Quickly build a chain
You can use a script to quickly build a one-node chain or four-node chain for testing.
```
cd ${WORKSPACE}/scripts

# build a one-node chain
./platonectl.sh one

# build a four-node chain
# ./platonectl.sh four
```

After a chain was built successfully, the data of the node will be generated in the directory `${WORKSPACE}/data/`. The log file can be used to watch the status of the node. For example, the node-0's log is located in the directory `${WORKSPACE}/data/node-0/logs/platone_log/`.

## Create a PlatONE account
Before sending a transaction, you need to create an account.

```
cd ${WORKSPACE}/../../cmd/SysContracts/

./script/create_accout.sh --ip 127.0.0.1 --rpc_port 6791
```

By this command you can create a new account, you need to enter an account password when creating, to facilitate unlocking you account.
The results are as follows, the account's address is recorded in the file `./config.json`:

```
nodeid: 127.0.0.1
rpc_port: 6791

###########################################
####       Create an account           ####
###########################################

Input account passphrase.
passphrase: your_phrase 
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
## Unlock account
Once a account does not send transactions for a long time, or the node is restarted, then the account will be locked and can not send transactions. At that time, you need to make use of the following command to unlock the account again.

```
cd ${WORKSPACE}/../../cmd/SysContracts/
./script/unlock_accout.sh \
            --account 0x08e7988e60ab5aa49d1f7aa9435ac91b9fcf772c \
            --phrase "your_phrase" \
            --ip 127.0.0.1 \
            --rpc_port 6791
```

## Create and compile user's contract
Start by creating a default contract with the following command, which contains two basic methods, `setName` and `getName`, on which the user can write contract.

```
cd ${WORKSPACE}/../../cmd/SysContracts/
# create default contract
./script/autoprojectForApp.sh  . my_contract

# compile the contract
./script/autoprojectForApp.sh  .
```
The generated contract source file is `${WORKSPACE}/../../cmd/SysContracts/appContract/my_contract/my_contract.cpp`.

## Deploy contract

```
cd ${WORKSPACE}/../../cmd/SysContracts/build

# ctool is used for deploying contracts and sending transactions
cp ${WORKSPACE}/bin/ctool .  

# deploy contract
./ctool deploy --abi appContract/my_contract/my_contract.cpp.abi.json --code appContract/my_contract/my_contract.wasm --config ../config.json
```

the result is as follows:
```
trasaction hash: 0x2f3a868d8ceb60804a830e5fc35611e3ae22ccb6ef298830acd84aba41f6dd99
contract address: 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206
```
## Invoke contract

invoke the `setName` method in contract

```
./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/my_contract/my_contract.cpp.abi.json   --config ../config.json  --func "setName" --param "wxbc"  

 request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xdb8800000000000000028c696e766f6b654e6f746966798477786263","txType":2}] 

 response json：{"jsonrpc":"2.0","id":1,"result":"0xeb3680c65b393952a07ec590cef7b19fc87877a9fbb70c8e16797a97ed4cfaeb"}
```

invoke the `getName` method in contract

```
./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/my_contract/my_contract.cpp.abi.json   --config ../config.json  --func getName

request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xd1880000000000000002876765744e616d65","txType":2},"latest"] 

response json：{"jsonrpc":"2.0","id":1,"result":"0x000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000047778626300000000000000000000000000000000000000000000000000000000"}
 
result: wxbc
```
## Interact with nodes through console
You can use `http` to enter the PlatONE console.
```
cd  ${WORKSPACE}/bin

./platone attach http://127.0.0.1:6791
```
the result is as follows:
```
Welcome to the PlatONE JavaScript console!

instance: PlatONEnetwork/platone/v0.2.0-stable-1b13ff73/linux-amd64/go1.12.4
coinbase: 0x938c231429f5ab34244618fe0dc7380e319b470e
at block: 1557 (Tue, 18 Jun 2019 16:29:12 CST)
 datadir: /home/gexin/PlatONE-Workspace/chain/PlatONE_linux/data/node-1
 modules: admin:1.0 eth:1.0 net:1.0 personal:1.0 rpc:1.0 web3:1.0

> 
```
## Look up transaction receipts

```
> eth.getTransactionReceipt("0x2f3a868d8ceb60804a830e5fc35611e3ae22ccb6ef298830acd84aba41f6dd99")
```

the result is as follows:

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
