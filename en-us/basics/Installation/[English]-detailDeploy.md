# PlatONE Manually Deploy Command Tutorial

The first chapter of this document describes the environment that PlatONE needs to run. It is recommended that users pre-install the software in this chapter before proceeding with the subsequent chapters. The second chapter explains in detail how PlatONE is compiled and deployed, and gives specific details. The steps are explained in Section 3, which demonstrates how to initialize and start PlatONE; Chapter 4 explains the deployment and invocation of system contracts.

# 1. Preparations

The warehouse address for PlatONE git is:

Enter the following command from the command line, or set it as an environment variable.

```bash
REPO_ADDR_HTTP="https://github.com/PlatONEnterprise/PlatONE-Go.git"
REPO_ADDR_GIT="git@github.com:PlatONEnterprise/PlatONE-Go.git"
```

## 1.1. Environment

* gcc 7.3+
* cmake 3.10+
* go 1.11.4+

# 2. Complie and Build

## 2.1. Download Source Code

There are two ways to download the source code of PlatONE.

- The first method: use `https` protocol

``` bash
git clone --recursive  ${REPO_ADDR_HTTP}
```
if you use `https` protocol to download the source code, you need to input your "username" and "password".

- The second method: use `git` protocol.

``` bash
git clone --recursive ${REPO_ADDR_GIT}
```

## 2.2. Compile

``` console
cd PlatONE-Go; 
make all
```

# 3. Init Node and Start

## 3.1. Init Node
### 3.1.1. Generate Account and Key Pair

- Configure environment variable. Enter the folder PlatONE-Go/build/bin

  ```cons
  export PATH=${PATH}:${PWD}
  ```

- Generate a new **user account**, you need the password to unlock your account. In our case, the account password is 0.

  ```console
  $ ./platone --datadir ./data account new
  
  INFO [01-09|17:25:14.269] Maximum peer count                       ETH=50 LES=0 total=50
  Your new account is locked with a password. Please give a password. Do not forget this password.
  Passphrase: 
  Repeat passphrase: 
  Address: {60208c048e7eb8e38b0fac40406b819ce95aa7af}
  ```

- Look up account

  ```console
  $ ll data/keystore/
  
  -rw------- 1 wxuser wxuser 491 Jan  9 17:25 UTC--2019-01-09T09-25-28.487164507Z--60208c048e7eb8e38b0fac40406b819ce95aa7af
  ```

- Enter the folder `PlatONE-Go/build/bin` and generate **node key pair**:  

  ```shell
  $ ./ethkey genkeypair
  
  Address   :  0xC71433b47f1b0053f935AEf64758153B24cE7445  # node address
  PrivateKey:  b428720a89d003a1b393c642e6e32713dd6a6f82fe4098b9e3a90eb38e23b6bb # node privatekey
  PublicKey: 68bb049008c7226de3188b6376127354507e1b1e553a2a8b988bb99b33c4d995e426596fc70ce12f7744100bc69c5f0bce748bc298bf8f0d0de1f5929850b5f4 # node publickey
  ```

- Store PrivateKey in file  `./data/platone/nodekey`, where the "PrivateKey" is generated previously 

  ```console
  $ mkdir -p ./data/platone
  
  $ echo "b428720a89d003a1b393c642e6e32713dd6a6f82fe4098b9e3a90eb38e23b6bb" > ./data/platone/nodekey
  
  $ cat ./data/platone/nodekey
  
  $ sudo updatedb
  
  $ locate nodekey
  
  /home/wxuser/work/golang/src/github.com/PlatONEnetwork/PlatONE-Go/build/bin/data/platone/nodekey
  ```

### 3.1.2. Generate cnsProxy Bytecode File

- go to path `PlatONE-Go/cmd/SysContracts` and execute script to generate makefile

  ```
  ./script/build_system_contracts.sh
  ```


Now, the "systemContract" folder which stores compiled files will be generate in path `build`

- go to path `PlatONE-Go/cmd/SysContracts/build/systemContract/cnsProxy` and execute "ctool" to get the bytecode

  ```
  ctool codegen --abi cnsProxy.cpp.abi.json --code cnsProxy.wasm
  ```

the bytecode would be put into "genesis.json" in 3.1.3.

### 3.1.3. Configure Inital File

- Generate genesis.json:
  - validatorNodes, suggestObserverNodes：the enode format is ‘enode://publicKey@ip:p2p_port’, and the publicKey that is promoted in section` 4) `of 3.1.1 needs to replace the publicKey in the enode. Ip and p2p_port can be customized according to the situation.
  - alloc: assign an amount to the account address. The user account address is generated in the `2)` summary of Chapter 3.1.1.
  - system management contract: 0x0000000000000000000000000000000000000011. This is a fixed address
  - code: cnsProxy bytecode generated in 3.1.2.

```json
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
      "code": "cnsProxy-bytes"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

### 3.1.4. Inital Platone
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

## 3.2. Start Node
```console
$ platone --identity "platone" --datadir ./data --port 16789 --rpcaddr 0.0.0.0 --rpcport 6789 --rpcapi "db,eth,net,web3,admin,personal" --rpc --nodiscover --nodekey "./data/platone/nodekey" --verbosity 4 --wasmlog ./wasm.log --bootnodes "enode://68bb049008c7226de3188b6376127354507e1b1e553a2a8b988bb99b33c4d995e426596fc70ce12f7744100bc69c5f0bce748bc298bf8f0d0de1f5929850b5f4@127.0.0.1:16789"

(Attention：--verbosity 4 will print "wasm log", command "--wasmlog" can specify which file to output the log to. option "--bootnodes", used to specify the nodes in the suggestObserverNodes in the genesis.json file )

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

### 3.2.1. platone log related startup parameters

When starting platone, specify the `--moduleLogParams` parameter to write the log partition of the platone to the file.

```bash
--moduleLogParams '{"platone_log": ["/"], "__dir__": ["../../logs"], "__size__": ["67108864"]}'
```

Parameter Description:

- `platone_log`: Specifies the log of which module in the output platone. For example, `"platone_log": ["/consensus", "/p2p"]`, only the logs printed in the consensus module and the p2p module are output.
  - `"platone_log": ["/"]` It means that the logs of all modules are output.
- `__dir__`: The directory location of the specified log output.
- `__size__`: Specifies the block size of the log write file.

Over time, the log files will accumulate and it is recommended to mount or periodically delete them.

More platone startup parameters, you can execute the following command to view.

```bash
platone -h 
```

### 3.3 Reset Node

When you have to reset your platone node, make sure platone process has been killed, and then delete the `data` folder. (If you need to follow the next chapter steps, this process needs to be omitted first)

```console
$ rm -rf data/platone
```

After that you can init your node again.

# 4. System contract deployment

## 4.1 Generating ctool.json

Go to the `PlatONE-Go/cmd/SysContracts/build/systemContract` directory and make sure the platone is started. Use vi to create the ctool.json file and write the following. Replace NODE-IP, RPC-PORT, DEFAULT-ACCOUNT in the template below according to the node started at this time.

```json
$ vi ctool.json

{
  "url": "http://NODE-IP:RPC-PORT",
  "gas": "0x0",
  "gasPrice": "0x0",
  "from": "0xDEFAULT-ACCOUNT"
}
```

- NODE-IP: The ip option set when the node starts.
- RPC-PORT: Node startup is the set rpc_port port.
- DEFAULT-ACCOUNT: User account created in subsection 2 of 3.1.1.

## 4.2 Deploying System Contracts

Before deploying the system contract, you need to unlock the account address of the deployment contract, first enter the console and unlock the user account.

```bash
$ platone attach http://NODE-IP:RPC-PORT
Welcome to the PlatONE JavaScript console!

Instance: PlatONEnetwork/platone/v0.2.0-stable-56ea60ae/linux-amd64/go1.11.4
Coinbase: 0x0fbd63b374002cb15aca95202fe10b63bda3fdcb
At block: 4012 (Tue, 27 Aug 2019 10:54:40 CST)
 Datadir: /home/wxuser/wywforfun/PlatONE-Go/build/bin/data
 Modules: admin:1.0 eth:1.0 net:1.0 personal:1.0 rpc:1.0 web3:1.0

>
```

Then unlock the user account, you need to enter the password corresponding to the account.

```bash
>personal.unlockAccount("DEFAULT-ACCOUNT")
Unlock account DEFAULT-ACCOUNT
Passphrase:
True

```

Then you can exit the console for contract deployment.

- The values of NODE-IP, RPC-PORT, and DEFAULT-ACCOUNT must be the same as those set in ctool.json in Chapter 4.1.

Go to the `PlatONE-Go/cmd/SysContracts/build/systemContract` directory

```bash
#Deploy cnsManager system contract
ctool deploy --config ctool.json --code cnsManager/cnsManager.wasm --abi cnsManager/cnsManager.cpp.abi.json
#Deploy paramManager system contract
ctool deploy --config ctool.json --code paramManager/paramManager.wasm --abi paramManager/paramManager.cpp.abi.json
#Deploy userManager system contract
Ctool deploy --config ctool.json --code userManager/userManager.wasm --abi userManager/userManager.cpp.abi.json
#Deploy userRegister system contract
ctool deploy --config ctool.json --code userRegister/userRegister.wasm --abi userRegister/userRegister.cpp.abi.json
#Deploy the roleManager system contract
ctool deploy --config ctool.json --code roleManager/roleManager.wasm --abi roleManager/roleManager.cpp.abi.json
#Deploy the roleRegister system contract
ctool deploy --config ctool.json --code roleRegister/roleRegister.wasm --abi roleRegister/roleRegister.cpp.abi.json
#Deploy nodeManager system contract
ctool deploy --config ctool.json --code nodeManager/nodeManager.wasm --abi nodeManager/nodeManager.cpp.abi.json
#Deploy nodeRegister system contract
ctool deploy --config ctool.json --code nodeRegister/nodeRegister.wasm --abi nodeRegister/nodeRegister.cpp.abi.json
```

**System Contract Description**

* cnsManager: Maps the contract name to contract information (contract address and version, etc.).
* paramManager: can dynamically follow some system parameters during the new PlatONE run.
* userManager: Manage users and user information in PlatONE.
* userRegister: Provides the ability to apply for registration as a platform user.
* roleManager: The permissions of the roles and roles of the management platform.
* roleRegister: Provides the ability to apply for registration to get the platform role.
* nodeManager: Management node and node information.
* nodeRegister: Provides the ability to apply for registration as a platform node (not necessary).

### 4.3 System Contract Call

This section will demonstrate how system contracts are invoked by adding nodes, viewing all nodes, and updating node information.

First enter the `PlatONE-Go/cmd/SysContracts/build/systemContract` directory

**Add Nodes**

```bash
ctool cnsInvoke --cns "__sys_NodeManager" --config ctool.json --abi nodeManager/nodeManager.cpp.abi.json --func add --param '{"name":"test","type":0," publicKey ":" 68bb049008c7226de3188b6376127354507e1b1e553a2a8b988bb99b33c4d995e426596fc70ce12f7744100bc69c5f0bce748bc298bf8f0d0de1f5929850b5f4 "," desc ":" "," externalIP ":" 127.0.0.1 "," internalIP ":" 127.0.0.1 "," rpcPort ": 6792," p2pPort ": 16792," owner ": "", "status": 1} '
```

Where option `--param` part parameters:

* name: The name of the node. In a node management contract, the node name must be unique.
* type: node type. Type is 0 for the observer node and type 1 for the consensus node. When adding a node, the type is set to 0. After the node runs stably, you can update the node information setting type to 1, and let the node participate in the consensus.
* publicKey: The node public key. The node public key is available in section 4 of section 3.1.1.
* status: Node status. When the status is set to 1, it indicates that the node is in a normal state and can be connected to other nodes. When the status is set to 2, it indicates that the node is in the deleted state, and the connection with other nodes will be disconnected.

**View all nodes**

After adding the node, run the following command to view all the information about all the added nodes in the system contract at this time:

```bash
ctool cnsInvoke --cns "__sys_NodeManager" --config ctool.json --abi nodeManager/nodeManager.cpp.abi.json --func "getAllNodes"
```

**Update node information**

When you need to follow the new node information, you need to call the node management contract update method. If you convert a normal node to a consensus node, the command is as follows:

```bash
ctool cnsInvoke --cns "__sys_NodeManager" --config ctool.json --abi nodeManager/nodeManager.cpp.abi.json --func "update" --param "test" --param '{"type":1}'
```

In this command, the first param option specifies the name of the node to be updated, and the second param option specifies the information of the node that needs to be updated.


