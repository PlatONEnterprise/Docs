# PlatONE Contracts Tutorial


The smart contract system used in PlatONE is completely different from traditional smart contracts, characteristics are as follows:

- distributed service platform under a decentralized topology
- Oriented computing, FaaS (Functions as a Service)
- The underlying virtual machine BCWasm
- The Wasm contracts use C++ syntax
- Contract built-in library (PlatONE lib)


## 1. Setting Up Wasm Contract Development Environment

Get the source code of  PlatONE  and compile it

```
wget https://github.com/PlatONEnterprise/PlatONE-Go/releases/download/v0.9.0/BCWasm_linux_release.v0.9.0.tar.gz

# Enter the BCWasm directory and define the path:
cd BCWasm
export CONTRACTSSPACE=${PWD}
```

The directory `${CONTRCATSSPACE}` contains the materials needed for contract compilation. The contract source directory is `${CONTRCATSSPACE}/appContract`

Get the latest stable release of PlatONE and unzip it. Go to the PlatONE_linux directory obtained by decompression, and then define the path:
```
wget https://github.com/PlatONEnterprise/PlatONE-Go/releases/download/v0.9.0/PlatONE_linux_v0.9.0.tar.gz

tar -xzvf PlatONE_linux_v0.9.0.tar.gz

cd PlatONE_linux
export BIN_PATH=${PWD}/bin
```
⚠Make sure ${BIN_PATH} is set in the current working shell for the environment variable to take effect.

## 2. Starting Project and Compile Contract


```shell
cd ${CONTRCATSSPACE}

# 1. construct user's contract 
./script/autoprojectForApp.sh  . my_contract

# 2. construct project
./script/autoprojectForApp.sh  .

# 3. compile contract
cd build
make my_contract
```

### 2.1. Construct Project

- create a new directory `my_contract` in path `${CONTRCATSSPACE}/appContract`
- create `build` directory in path `${CONTRCATSSPACE}` and then compile the contract

Now the structure of  `build` directory is as follows.

```
├── appContract
│   ├── appDemo
│   │   ├── appDemo.bc
│   │   ├── appDemo.cpp.abi.json ## contract abi file
│   │   ├── appDemo.cpp.bc
│   │   ├── appDemo.s
│   │   ├── appDemo.wasm         ## wasm bytecode of contract
│   │   ├── appDemo.wast
│   │   ├── CMakeFiles
│   │   ├── cmake_install.cmake
│   │   └── Makefile
│   ├── CMakeFiles
│   ├── cmake_install.cmake
│   ├── Makefile
│   └── my_contract
│       ├── CMakeFiles
│       ├── cmake_install.cmake
│       ├── Makefile
│       ├── my_contract.cpp.abi.json ## contract abi file
│       ├── my_contract.bc
│       ├── my_contract.cpp.bc
│       ├── my_contract.s
│       ├── my_contract.wasm
│       └── my_contract.wast
├── CMakeCache.txt
├── CMakeFiles
├── cmake_install.cmake
└── Makefile
```

- The contract file will generate the following source code by default in the script `autoprojectForApp.sh`, and you can write your own contract based on this.

```cpp
//auto create contract
#include <stdlib.h>
#include <string.h>
#include <string>
#include <bcwasm/bcwasm.hpp>

namespace demo {
class my_contract : public bcwasm::Contract
{
    public:
    my_contract(){}

    /// parent class implementation: bcwasm:: virtual function of Contract
    /// this function is called only once when the contract is first released
    void init() 
    {
        bcwasm::println("init success...");
    }
    /// define event.
    /// BCWASM_EVENT(eventName, arguments...)
    BCWASM_EVENT(setName, const char *)
    
    public:
    void setName(const char *msg)
    {
        // define state
        bcwasm::setState("NAME_KEY", std::string(msg));
        // print log
        // return event
        BCWASM_EMIT_EVENT(setName, "std::string(msg)");
    }
    const char* getName() const 
    {
        std::string value;
        bcwasm::getState("NAME_KEY", value);
        // read contract data and return
        return value.c_str();
    }
};
}
// the function below will generate an ABI file for external calls
BCWASM_ABI(demo::my_contract, setName)
BCWASM_ABI(demo::my_contract, getName)
```



### 2.2. Compile Contract

- go to `build` directory and compile the contract

```
cd ${CONTRCATSSPACE}/build
make my_contract
```

- generate files as follows

```
├build/ 
├-appContact/
│   └── my_contract
│       ├── CMakeFiles
│       ├── cmake_install.cmake
│       ├── Makefile
│       ├── my_contract.cpp.abi.json ## contract abi file
│       ├── my_contract.bc
│       ├── my_contract.cpp.bc
│       ├── my_contract.s
│       ├── my_contract.wasm
│       └── my_contract.wast
```

**Main documents:**

- `.bc`: Linked standard library LLVM bytecode files; including all dependent libraries
- `.json`: Contract interface description file 
- `.s`: Assembly file
- `.wasm`: Contract compiled binary file, bytecode instruction set
- `.wast`: instruction file that can be read and understood after the contract is compiled
- `.cpp.bc`: LLVM bytecode file, containing only the contract code itself

my_contract.wast and my_contract.cpp.abi.json will be used for contract deployment.


## 3. Deploy and Invoke Contract

### 3.1. Generate Contract Account

```
cd ${CONTRCATSSPACE}/build
../script/create_accout.sh --ip 127.0.0.1 --rpc_port 6791
```

create a new account with inputting a password which will be used for unlocking your account. Besides a `config.json` file will be generated in the current directory and this file will be used for invoking contract. What's more, this file records the node port and accounts that have sent transactions. The test result is as follows:

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

Profile field description:

- `url`: PlatONE node public JSON-RPC address 
- `from`: Address of account that is deploying the contract

Once a account does not send transactions for a long time, or the node is restarted, then the account will be locked and can not send transactions. At that time, you need to make use of the following command to unlock the account again.

```
../script/unlock_account.sh \
             --account 0x08e7988e60ab5aa49d1f7aa9435ac91b9fcf772c \
            --phrase "your_phrase" \
            --ip 127.0.0.1 \
            --rpc_port 6791
```

### 3.2. Deploy Contract

```shell
cd ${CONTRCATSSPACE}/build

# ctool is used for deploying contracts and sending transactions
cp  ${BIN_PATH}/ctool  .

# deploy contract
./ctool deploy --abi appContract/my_contract/my_contract.cpp.abi.json --code appContract/my_contract/my_contract.wasm --config ./config.json
```

the result is as follows:

```
trasaction hash: 0x2f3a868d8ceb60804a830e5fc35611e3ae22ccb6ef298830acd84aba41f6dd99
contract address: 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206
```

ctool tutorial : deploy contract and invoke contract

```shell
# deploy contract
./ctool                                                     \
    deploy                                                  \
    --abi   contract.cpp.abi.json                           \
    --code  contract.wasm                                   \
    --config ./config.json

# invoke contract
./ctool                                                     \
    invoke                                                  \
    --abi   contract.cpp.abi.json                           \
    --addr  "contract address： 0x1234"                            \
    --config ./config.json                                  \
    --func  "func name"                                       \
    --param  "the first parameter"                          \
    --param  "the second parameter"                                     
```

- **Please note**: If the configuration file path is not explicitly set on the command line, a file with the name `config.json` will be read from the current working path. 
- **Please note**: To deploy a contract to the PlatONE network, you need to connect to a node and make sure that the account used has been unlocked and that there is no timeout during deployment.Execute git-bash.exe to open the git-bash window.
- **Please note**: If the command cannot be executed, make sure the script has execute permissions and can be authorized with the command: `chmod +x ctool`.

### 3.3. Invoke Contract

invoke  `setName`  in Contract

```
#Address needs to be changed to the address generated in the previous section

./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/my_contract/my_contract.cpp.abi.json   --config ./config.json  --func "setName" --param "wxbc"  

 request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xdb8800000000000000028c696e766f6b654e6f746966798477786263","txType":2}] 
 response json：{"jsonrpc":"2.0","id":1,"result":"0xeb3680c65b393952a07ec590cef7b19fc87877a9fbb70c8e16797a97ed4cfaeb"}
```

invoke  `getName`  in contract

```json
./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/my_contract/my_contract.cpp.abi.json   --config ./config.json  --func getName

request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xd1880000000000000002876765744e616d65","txType":2},"latest"] 
response json：{"jsonrpc":"2.0","id":1,"result":"0x000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000047778626300000000000000000000000000000000000000000000000000000000"}
 
result: wxbc
```

## 4. Interact with Nodes Through Console

### 4.1. Interact with Nodes

```
cd ${BIN_PATH}

./platone attach http://127.0.0.1:6791

Welcome to the PlatONE JavaScript console!

instance: PlatONEnetwork/platone/v0.2.0-stable-1b13ff73/linux-amd64/go1.12.4
coinbase: 0x938c231429f5ab34244618fe0dc7380e319b470e
at block: 1557 (Tue, 18 Jun 2019 16:29:12 CST)
 datadir: /home/gexin/PlatONE-Workspace/chain/PlatONE_linux/data/node-1
 modules: admin:1.0 eth:1.0 net:1.0 personal:1.0 rpc:1.0 web3:1.0

> 
```

We can interact with nodes through the command above which can be used to look up transactions 

### 4.2. look up transaction receipts

```js
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

## 5. Appendix-BCWasm Contract Coding Tutorial

BCWasm is a wasm contract development tool and the corresponding path is `${CONTRCATSSPACE}`

### 5.1. BCWasm contract coding standards:

(1) Define contract by yourself must inherit **bcwasm::Contract** class and implement the init() method. The contract must implement the init virtual function of the parent contract. This function is executed only when the contract is first released. This method is similar to the constructor in the solidity contract.
```
class my_contract : public bcwasm::Contract
{
    public:
    my_contract(){}

    /// Implement the parent class: virtual function of bcwasm::Contract
    /// This function is executed when the contract is first released, only once.
    void init() 
    {
        bcwasm::println("init success...");
    }
}
```

(2) Contract external interfaces need to be exported by BCWASM_ABI macro, such as BCWASM_ABI(demo::FirstDemo, getName).
```
BCWASM_ABI(demo::my_contract, getName)
```

(3) Contract interfaces for external queries must be declared as a const member function of the contract class. Otherwise, there is no error in compiling, but the external calls to the contract interface will return null.
```
const char* getName() const 
{
    std::string value;
    bcwasm::getState("NAME_KEY", value);
    // Read contract data and return
    return value.c_str();
}
```

(4) The persistence process of the general class: the related class is instantiated. When the system performs gc, the destructor of the class is called. The destructor internally calls the flush() function and calls bcwasm::setState() to complete the persistence.

```
// Persistence
bcwasm::setState("NAME_KEY", std::string(msg));

// Inquire
std::string value;
bcwasm::getState("NAME_KEY", value);
```


(5) The current contract external interface only supports the following data types

> char/char* /const char*/char[]  
> unsigned \_\_int128/\_\_int128/uint128_t  
> unsigned long long/uint64_t  
> unsigned long/uint32_t  
> unsigned short/uint16_t  
> unsigned char/uint8_t  
> long long/int64_t  
> long/int32_t/int  
> short/int16_t  
> char/int8_t  
> void  

(6) To persist a custom data structure, you need to add the BCWASM_SERIALIZE macro to the structure declaration. The specific usage is as follows:

```C++
// Defines a Transaction structure with three members, serializing all three members when the structure is serialized
struct Transaction {  
	Address destination;  
	Address from;  
	int64_t value;  
	BCWASM_SERIALIZE(Transaction, (destination)(from)(value))  
};
```

(7) The platone contract library is undefined for u32 and fixed-length array bytesN, which can now be replaced with uint32_t and char[] arrays respectively.

(8) In the implementation of the contract external interface query method, if the function returns a string (such as by calling string c_str () method), you need to newly apply (malloc) a piece of memory inside the function, and copy the string to this new memory, because the memory is managed by the BCWasm virtual machine, there is no memory leak.When returning a string type, it can be implemented using the `RETURN_CHARARRAY` macro, which is defined as follows

```C++
#define RETURN_CHARARRAY(src, size) \
do \
{ \
    char *buf = (char *)malloc(size); \
    memset(buf, 0, size); \
    strcpy(buf, src); \
    return buf; \
} \
while(0)
```

(9) The u256 type conversion string type in the built-in library of the wasm contract needs to be called as follows. 

```
u256value.convert_to<std::string>()
```

(15) Macros BCWASM_EVENT and BCWASM_EMIT_EVENT provide direct support for contract events.

```
/// Define Event.
/// BCWASM_EVENT(eventName, arguments...)
BCWASM_EVENT(setName, const char *)

/// Trigger Event
BCWASM_EMIT_EVENT(setName, "std::string(msg)");
```