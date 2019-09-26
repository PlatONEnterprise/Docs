# PlatONE Contracts Tutorial


The smart contract system used in PlatONE is completely different from traditional smart contracts, characteristics are as follows:

- distributed service platform under a decentralized topology
- Oriented computing, FaaS (Functions as a Service)
- The underlying virtual machine BCWasm
- The Wasm contracts use C++ syntax
- Contract built-in library (PlatONE lib)


## 1. Setting Up Wasm Contract Development Environment

Get the latest stable release of BCWasm and unzip it. Go to the BCWasm directory obtained by decompression, and then define the path:

```
cd BCWasm

export CONTRCATSSPACE=${PWD}
```

Get the latest stable release of PlatONE and unzip it. Go to the PlatONE_linux directory obtained by decompression, and then define the path:
```
cd PlatONE_linux

export BINSPACE=${PWD}/bin
```

The material required for contract compilation is included in the $CONTRCATSSPACE directory, and its subdirectory functions are as follows:

The contract source directory appContract, by default has an appDemo contract project

## 2. Starting Project and Build Contract

```shell
# install dependency tools for Linux system
sudo apt install cmake g++ maven
```

```shell
cd ${CONTRCATSSPACE}

# 1. construct user's contract 

./script/autoprojectForApp.sh  . my_contract

# 2. construct project
./script/autoprojectForApp.sh  .

# 3. build contract
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

```c++
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
cd build

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

firstdemo.wast and firstdemo.cpp.abi.json will be used for contract deployment.

**Attention**：the following error log is expected during compilation.

```
Could not auto-detect compilation database for file "/workspace/appContact/my_contract/my_contract.cpp"
No compilation database found in /workspace/appContact/my_contract or any parent directory
json-compilation-database: Error while opening JSON database: No such file or directory
```

# 

## 3. Deploy and Invoke Contract

### 3.1. Generate Contract Account

```shell
cd build

../script/create_accout.sh --ip 127.0.0.1 --rpc_port 6791
```

create a new account with inputting a password which will be used for unlocking your account. Besides a `config.json` file will be generated in the current directory and this file will be used for invoking contract. What's more, this file records the node port and accounts that have sent transactions. The test result is as follows:

```json
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

**Profile field description:**

- `url`: PlatONE node public JSON-RPC address 
- `from`: Address of account that is deploying the contract

Once a account does not send transactions for a long time, or the node is restarted, then the account will be locked and can not send transactions. At that time, you need to make use of the following command to unlock the account again.

```shell
../script/unlock_accout.sh \
            --account 0x08e7988e60ab5aa49d1f7aa9435ac91b9fcf772c \
            --phrase "your phrase"
            --ip 127.0.0.1 \
            --rpc_port 6791
```

### 3.2. Deploy Contract

```shell
cd build 

# ctool is used for deploying contracts and sending transactions
# deploy contract
./ctool deploy --abi appContract/my_contract/my_contract.cpp.abi.json --code appContract/my_contract/my_contract.wasm --config ./config.json
```

the result is as follows:

```json
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
    --addr  "contract address： 0x1234"                               \
    --config ./config.json                                  \
    --func  "func name"                                       \
    --param  "the first parameter"                                     \
    --param  "the second parameter"                                     
```

- **Please note**: If the configuration file path is not explicitly set on the command line, a file with the name `config.json` will be read from the current working path. 

- **Please note**: To deploy a contract to the PlatONE network, you need to connect to a node and make sure that the account used has been unlocked and that there is no timeout during deployment.
- **Please note**: If the command cannot be executed, make sure the script has execute permissions and can be authorized with the command: `chmod +x ctool`.

### 3.3. Invoke Contract

invoke the `setName` method in Contract

```json
./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/my_contract/my_contract.cpp.abi.json   --config ./config.son  --func "setName" --param "wxbc"  

 request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xdb8800000000000000028c696e766f6b654e6f746966798477786263","txType":2}] 

 response json：{"jsonrpc":"2.0","id":1,"result":"0xeb3680c65b393952a07ec590cef7b19fc87877a9fbb70c8e16797a97ed4cfaeb"}
```

invoke the `getName` method in contract

```json
./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/my_contract/my_contract.cpp.abi.json   --config ./config.json  --func getName

request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xd1880000000000000002876765744e616d65","txType":2},"latest"] 

response json：{"jsonrpc":"2.0","id":1,"result":"0x000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000047778626300000000000000000000000000000000000000000000000000000000"}
 
result: wxbc
```

## 4. Interact with Nodes Through Console

### 4.1. Interact with Nodes

```shell
cd ${BINSPACE}

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

BCWasm is a wasm contract development tool and the corresponding path is `~/PlatONE-Go/cmd/SysContracts/`

### 5.1. BCWasm contract coding standards:

(1) Define contract by yourself must inherit **bcwasm::Contract** class

(2) Contract external interfaces need to be exported by BCWASM_ABI macro, such as BCWASM_ABI(demo::FirstDemo, invokeNotify)

(3) Contract interfaces for external queries must be declared as a const member function of the contract class. Otherwise, there is no error in compiling, but the external calls to the contract interface will return null.

(4) For the data structure encapsulated by the StorageType template, since the template has been overloaded with the dereference operator which can obtain the data structure in the template.

(5) The persistence process of the general class: the related class is instantiated. When the system performs gc, the destructor of the class is called. The destructor internally calls the flush() function and calls bcwasm::setState() to complete the persistence.

(6) The Wasm contract built-in library provides two ways for data persistence. The first method is to use the template provided in the library. The second method directly calls the bcwasm::setState() function, and the query calls bcwasm::getState() accordingly.

(7) All the templates in the Wasm contract built-in library end up almost  by calling the underlying bcwasm::setState() to complete the persistence operation.

(8) The custom contract must implement the parent contract's init virtual function, which is executed only when the contract is first released. It can therefore be used to simulate the persistence of state variables in a solidity contract and complete the initialization.

(9) The current contract external interface only supports the following data types

```
char/char* /const char*/char[]  
unsigned __int128/__int128/uint128_t  
unsigned long long/uint64_t  
unsigned long/uint32_t  
unsigned short/uint16_t  
unsigned char/uint8_t  
long long/int64_t  
long/int32_t/int  
short/int16_t  
char/int8_t  
float/double  
void  
```

(10) To persist a custom data structure, you need to add the BCWASM_SERIALIZE macro to the structure declaration. The specific usage is as follows:

```C++
struct Transaction {  
	Address destination;  
	Address from;  
	int64_t value;  
	BCWASM_SERIALIZE(Transaction, (destination)(value))  
};
```

(11) The platone contract library is undefined for u32 and fixed-length array bytesN, which can now be replaced with uint32_t and char[] arrays respectively.

(12) In the implementation of the contract external interface query method, if the function returns a string (such as by calling string c_str () method), you need to newly apply (malloc) a piece of memory inside the function, and copy the string to This new memory, because the memory is managed by the BCWasm virtual machine, there is no memory leak.

(13) At present, platone lib only supports some keywords in the solidity language, and will support these keywords step by step in the future. The list of keywords that currently need to be supported is as follows:

| solidity contract | C++ contract |
| ----------------- | :----------: |
| require           |   require    |
| revert            |   require    |
| selfdestruct      |   require    |

(14) The u256 type conversion string type in the built-in library of the wasm contract needs to be called as follows. Otherwise, the deployment contract will report "panic: send transaction error, error:intrinsic gas too low":

```
u256value.convert_to<std::string>()
//need to modify the gasLimit of genesis.json and increase the gas value of the ctool configuration file.
```

(15) Macros BCWASM_EVENT and BCWASM_EMIT_EVENT provide direct support for contract events.

(16) Contract data isolation (account level)

```go
func (self *WasmStateDB) SetState(key []byte, value []byte)  {
	self.evm.StateDB.SetState(self.Address(), key, value)
}

func (self *WasmStateDB) GetState(key []byte) []byte {
	return self.evm.StateDB.GetState(self.Address(), key)
}
```

The underlying persistence of BCWasm calls the EVM's SetState(self.Address(), key, value) method, which means that data is isolated between different accounts.

(17) Deserialization rule

The underlying deserialization of BCWasm is implemented by getState, the code is as follows:

```C++
template <typename KEY, typename VALUE>
inline size_t getState(const KEY &key, VALUE &value) {
    std::vector<char> vecKey(pack_size(key));
    DataStream<char*> keyStream(vecKey.data(), vecKey.size());
    keyStream << key;
    size_t len = ::getStateSize((const byte*)vecKey.data(), vecKey.size());
    if (len == 0){ return 0; }
    std::vector<char> vecValue(len);
    ::getState((const byte*)vecKey.data(), vecKey.size(), (byte*)vecValue.data(), vecValue.size());

    DataStream<char*> valueStream(vecValue.data(), vecValue.size());
    valueStream >> value;
    return len;
}
```

From the above code, it is seen that the deserialization itself is filled from the valueStream by byte. The format of the padding follows the data structure of the value itself (including the data offset of each member). You can customize the serialization and deserialization format by BCWASM_SERIALIZE, which is to select which members in the structure are on the chain and which are not on the chain. The code is as follows:

```C++
#define BCWASM_SERIALIZE( TYPE,  MEMBERS ) \
 template<typename DS> \
 friend DS& operator << ( DS& ds, const TYPE& t ){ \
    return ds BOOST_PP_SEQ_FOR_EACH( BCWASAM_REFLECT_MEMBER_OP, <<, MEMBERS );\
 }\
 template<typename DS> \
 friend DS& operator >> ( DS& ds, TYPE& t ){ \
    return ds BOOST_PP_SEQ_FOR_EACH( BCWASM_REFLECT_MEMBER_OP, >>, MEMBERS );\
 }
```

example:

```C++
struct onchainData{
    unsigned long member1;
    unsigned long member2;
	unsigned long member3;
	std::vector<string> arr1;
    BCWASM_SERIALIZE(onchainData, (member1)(member2)(arr1));
};
```

For sequences and deserialization of arrays that currently only support vector<> types within members, c arrays such as char[] are not supported.

(18) Definition and use of RETURN_CHARARRAY 

- definition

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

- usage example:

```C++
const char *getName() const
{
    std::string name;
    platone::getState("NAME_KEY", name);
    size_t size = name.size();
    RETURN_CHARARRAY(name.c_str(), size + 1);
}
```

(19) The BCWasm contract does not support C++14 regular expression functions.

(20) BCWasm lib itself does not support retrieving block-related information, but the wasm virtual machine provides the following interfaces through state.hpp to retrieve commonly used block information, such as timestamps:

```C++
extern "C" {
    int64_t gasPrice();
    void blockHash(int64_t num,  uint8_t[32]);
    uint64_t number();
    uint64_t gasLimit();
    int64_t timestamp();
    void coinbase(uint8_t hash[20]);
    void balance(uint8_t amount[32]);
    void origin(uint8_t hash[20]);
    void caller(uint8_t hash[20]);
    void callValue(uint8_t val[32]);
    void address(uint8_t hash[20]);
    void sha3(const uint8_t *src, size_t srcLen, uint8_t *dest, size_t destLen);
    
    int64_t getCallerNonce();
    int64_t callTransfer(const uint8_t* to, size_t toLen, uint8_t amount[32]);
}
```

(21) When you use ctool to specify function parameters, the parameters need to be separated by commas, note that there can be no spaces in the middle.

(22) Currently the get method of the StroageType template is non-const type, the returned variables need to be stored (otherwise it will be automatically destructed), and the cosnt type get method needs to be supported in the future.

(23) getContractAddress(char *name, char *version), name + version version needs to be converted to a spliced string

(24) The wasm log log sometimes fails to print. You need to manually restart the platone node.

(25) The old version of the platone code for json serialization bugs has been fixed in the new version.

(26) By default, the WASM virtual machine allocates only 200 K of memory to the smart contract. The total amount of data manipulated by the smart contract cannot exceed this limit. Although it can increase the memory space of the virtual machine, it will slow down the initialization speed of the virtual machine, which will lead to the decrease of the throughput of the whole block chain. The following are the main ways to use memory:

> Data structures such as array, map and list in DB namespace.

> Bcwasm:: setState () method, bcwasm:: getState () method, malloc() method.
