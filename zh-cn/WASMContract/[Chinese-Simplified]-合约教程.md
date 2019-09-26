PlatONE智能合约体系完全不同于传统智能合约，主要特点：

- 位于PlatONE的分布式服务平台
- 面向计算，FaaS (Functions as a Service)
- 底层虚拟机BCWasm
- 合约开发语言C++
- 合约内置库（PlatONE lib）



## 1. 配置Wasm合约开发环境

获取BCWasm最新稳定版的发布包，并进行解压。进入解压获得的BCWasm目录，然后定义路径：
```
cd BCWasm

export CONTRCATSSPACE=${PWD}
```
获取PlatONE最新稳定版的发布包，并进行解压。进入解压获得的PlatONE_linux目录，然后定义路径：
```
cd PlatONE_linux

export BINSPACE=${PWD}/bin
```
在$CONTRCATSSPACE目录下包含了合约编译所需的材料，其子目录功能如下所示：


合约源码目录appContract， 默认有一个appDemo的合约工程

## 2. 构建项目及编译合约

```
# 依赖工具安装
# ubuntu下
sudo apt install cmake g++ maven
# centos下
sudo yum install cmake g++ maven
```

```
cd ${CONTRCATSSPACE}

# 1. 构建用户合约

./script/autoprojectForApp.sh  . my_contract

# 2. 构建工程
./script/autoprojectForApp.sh  .

# 3. 编译合约
cd build
make my_contract
```

### 2.1. 构建项目

第一步时，会在`${CONTRCATSSPACE}/appContract`下构建新的合约目录`my_contract`。

第二步时，会在`${CONTRCATSSPACE}`下创建`build`，并编译合约。

此时，`build`目录下的结构如下所示

```
├── appContract
│   ├── appDemo
│   │   ├── appDemo.bc
│   │   ├── appDemo.cpp.abi.json ## 合约的abi文件
│   │   ├── appDemo.cpp.bc
│   │   ├── appDemo.s
│   │   ├── appDemo.wasm         ## 合约的wasm字节码
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
│       ├── my_contract.cpp.abi.json ## 合约的abi文件
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

合约文件会通过脚本`autoprojectForApp.sh`默认生成如下的源码，可以在此基础上编写自己的合约。

```cpp
#include <stdlib.h>
#include <string.h>
#include <string>
#include <bcwasm/bcwasm.hpp>

namespace demo {
class my_contract : public bcwasm::Contract
{
    public:
    my_contract(){}

    /// 实现父类: bcwasm::Contract 的虚函数
    /// 该函数在合约首次发布时执行，仅调用一次
    void init() 
    {
        bcwasm::println("init success...");
    }
    /// 定义Event.
    BCWASM_EVENT(Notify, uint64_t, const char *)
    
    public:
    void invokeNotify(const char *msg)
    {    
        // 定义状态变量
        bcwasm::setState("NAME_KEY", std::string(msg));
        // 日志输出
        bcwasm::println("into invokeNotify...");
        // 事件返回
        BCWASM_EMIT_EVENT(Notify, 0, "Insufficient value for the method.");
    }
    const char* getName() const 
    {
        std::string value;
        bcwasm::getState("NAME_KEY", value);
        // 读取合约数据并返回
        return value.c_str();
    }
};
}
// 此处定义的函数会生成ABI文件供外部调用
BCWASM_ABI(demo::my_contract, invokeNotify)
BCWASM_ABI(demo::my_contract, getName)
```

### 2.2. 编译合约

进入`build`目录并编译该合约

```
cd build

make my_contract
```

生成文件如下

```
├build/ 
├-appContact/
│   └── my_contract
│       ├── CMakeFiles
│       ├── cmake_install.cmake
│       ├── Makefile
│       ├── my_contract.cpp.abi.json ## 合约的abi文件
│       ├── my_contract.bc
│       ├── my_contract.cpp.bc
│       ├── my_contract.s
│       ├── my_contract.wasm
│       └── my_contract.wast
```

主要文件简介：

- .bc 连接标准库后的LLVM字节码文件，含所有依赖库；
- .json 合约对应的接口描述文件；
- .s 汇编文件；
- .wasm 合约编译后的二进制文件，字节码指令集；
- .wast 合约编译后的可被认为读懂的指令文件；
- .cpp.bc ；LLVM 字节码文件，仅包含合约代码本身的；

其中，发布合约时需要用到的文件为firstdemo.wast和firstdemo.cpp.abi.json。

提示： 如果在编译中出现如下错误信息，请不要惊慌，正常输出。

```
Could not auto-detect compilation database for file "/workspace/appContact/my_contract/my_contract.cpp"
No compilation database found in /workspace/appContact/my_contract or any parent directory
json-compilation-database: Error while opening JSON database: No such file or directory
```

## 3. 部署及调用合约

### 3.1. 生成合约部署账户

```
cd build

../script/create_accout.sh --ip 127.0.0.1 --rpc_port 6791
```

该命令会创建一个新的账户，创建时需要输入一个账户密码，方便后续解锁。

除了新生成一个账户，该命令还会在当前目录创建一个`config.json`文件，该文件在调用合约时会用到。该文件记录节点的ip端口以及发送交易的账号。

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

配置文件字段简述：

- url PlatONE节点开放的JSON-RPC地址信息；
- `from` 发布合约者的账户地址。

长时间没用使用账户发送过交易，或者节点重启后，账户会被锁定，不能发送交易。需要使用如下命令，重新为账户解锁。

```
../script/unlock_account.sh 
             --account 0x08e7988e60ab5aa49d1f7aa9435ac91b9fcf772c \
            --phrase "your phrase"
            --ip 127.0.0.1 \
            --rpc_port 6791
```

### 3.2. 部署合约

```
cd build 

# ctool是用来部署合约及发送交易的工具。
# 部署合约
./ctool deploy --abi appContract/my_contract/my_contract.cpp.abi.json --code appContract/my_contract/my_contract.wasm --config ./config.json
```

结果:

```
trasaction hash: 0x2f3a868d8ceb60804a830e5fc35611e3ae22ccb6ef298830acd84aba41f6dd99
contract address: 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206
```

ctool工具使用方式主要有以下两种：

```shell
# 合约部署
./ctool                                                     \
    deploy                                                  \
    --abi   contract.cpp.abi.json                           \
    --code  contract.wasm                                   \
    --config ./config.json

# 合约调用
./ctool                                                     \
    invoke                                                  \
    --abi   contract.cpp.abi.json                           \
    --addr  "合约地址： 0x1234"                               \
    --config ./config.json                                  \
    --func  "方法名字"                                       \
    --param  "第1个参数"                                     \
    --param  "第2个参数"                                     
```

- 提示1： 如果命令行中未明确指明配置文件路径，则会在当前工作路径读取文件：config.json。
- 提示2： 发布合约到PlatONE网络，需要连接到节点，并保证发布合约的账户已进行了解锁操作，且没有超时。执行git-bash.exe打开git-bash窗口
- 提示3： 如果命令不能执行，请确保脚本具有执行权限，可使用命令：chmod +x ctool 进行授权。

### 3.3. 调用合约

调用合约的`setName方法`

```
#地址需要改成上节生成的地址

./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/my_contract/my_contract.cpp.abi.json   --config ./config.json  --func "setName" --param "wxbc"  


 request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xdb8800000000000000028c696e766f6b654e6f746966798477786263","txType":2}] 

 response json：{"jsonrpc":"2.0","id":1,"result":"0xeb3680c65b393952a07ec590cef7b19fc87877a9fbb70c8e16797a97ed4cfaeb"}
```

调用合约的`getName`方法：

```
#地址需要改成上节生成的地址

./ctool invoke --addr 0x2ee8d0545ebd20f9a992ff54cb0f21a153539206 --abi appContract/my_contract/my_contract.cpp.abi.json   --config ./config.json  --func getName

request json data：[{"from":"0x32ab0a20b589f40c7e3d6ee485a2404bb7269f87","to":"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206","gas":"0x0","gasPrice":"0x0","value":"","data":"0xd1880000000000000002876765744e616d65","txType":2},"latest"] 

response json：{"jsonrpc":"2.0","id":1,"result":"0x000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000047778626300000000000000000000000000000000000000000000000000000000"}
 
result: wxbc
```

## 4. 通过console与节点交互

### 4.1. 节点交互

```
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

通过上面命令可以与节点交互，用于查询相关信息

### 4.2. 查询交易receipt

```
#交易哈希值改成上节生成的
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

## 5. BCWasm合约编码教程

BCWasm为wasm合约开发工具，其对应路径为`~/PlatONE-Go/cmd/SysContracts/`

### 5.1. BCWasm合约编码规范：

1. 自定义合约必须继承bcwasm::Contract类。
2. 合约对外接口需要通过BCWASM_ABI宏导出比如BCWASM_ABI(demo::FirstDemo, invokeNotify)。
3. 合约对外查询接口，该接口必须声明为其所在合约类的const型成员函数，否则编译没有错误但是于外部调用该合约接口是返回数据为空。
4. 对于StorageType模板封装的数据结构，由于该模板已将解引用操作符进行了重载，所以可以通过该解引用操作符拿到该模板中的数据结构。
5. 一般类的持久化流程:相关类进行实例化赋值，当系统进行gc时会调用类的析构函数，析构函数内部会调用flush()函数进而调用bcwasm::setState()完成持久化。
6. Wasm合约内置库为数据的持久化提供了两种方式，方式一调用使用库中提供的模板，方式二直接调用bcwasm::setState()函数，相应地查询调用bcwasm::getState()。
7. Wasm合约内置库中的所有模板最终几乎都是通过调用底层bcwasm::setState()完成持久化的操作。
8. 自定义合约必须实现父合约的init虚函数，该函数在合约首次发布时执行，仅调用一次。因此可以用来模拟solidity合约中状态变量的持久化并完成初始化操作。
9. 当前合约对外接口仅支持以下数据类型：

> char/char* /const char*/char[]  
> unsigned __int128/__int128/uint128_t  
> unsigned long long/uint64_t  
> unsigned long/uint32_t  
> unsigned short/uint16_t  
> unsigned char/uint8_t  
> long long/int64_t  
> long/int32_t/int  
> short/int16_t  
> char/int8_t  
> float/double  
> void  

10. 对自定义数据结构进行持久化需要在结构体声明中加入BCWASM_SERIALIZE宏，具体用法如下：

```C++
struct Transaction {  
	Address destination;  
	Address from;  
	int64_t value;  
	BCWASM_SERIALIZE(Transaction, (destination)(value))  
};
```

11. platone合约库对u32和定长数组bytesN未定义，目前可以分别用uint32_t和char[]数组代替。

12. 在实现合约对外接口的查询方法是时，若函数返回的是字符串（比如通过调用string的c_str()方法），需要在该函数内部新申请（malloc）一段内存，并将该字符串copy到这段新的内存，由于该内存是由BCWasm虚拟机统一管理，故不存在内存泄露问题。

13. 目前platone lib只支持solidity语言中的部分关键字，未来将逐步完成对这些关键字的支持。目前需要支持的关键字列表如下：

    | solidity合约 | C++合约 |
    | ------------ | :-----: |
    | require      |  需要   |
    | revert       |  需要   |
    | selfdestruct |  需要   |

14. wasm合约内置库中的u256类型转换字符串类型需要进行如下调用,否则部署合约时会报错“panic: send transaction error ,error:intrinsic gas too low”：

> u256value.convert_to<std::string>()<br/>
> 同时需要修改genesis.json的gasLimit大小并且提高ctool的配置文件的gas数值。

15. 宏BCWASM_EVENT和BCWASM_EMIT_EVENT提供了对合约event的直接支持。
16. 合约数据隔离(账户级别)

```golang
func (self *WasmStateDB) SetState(key []byte, value []byte)  {
	self.evm.StateDB.SetState(self.Address(), key, value)
}

func (self *WasmStateDB) GetState(key []byte) []byte {
	return self.evm.StateDB.GetState(self.Address(), key)
}
```

BCWasm底层持久化调用的是EVM的SetState(self.Address(), key, value)方法，即不同账户间是数据隔离的。

17. 反序列化的规则

BCWasm底层反序列化是通过getState实现的，代码如下：

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

从以上代码看出反序列化本身就是从valueStream里逐byte的填充，填充的格式遵循value本身的数据结构（包含各成员的数据偏移）。可以通过BCWASM_SERIALIZE来订制序列化和反序列化的格式，即选择结构体里哪些成员上链，哪些不上链，代码如下

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

举例：

```C++
struct onchainData{
    unsigned long member1;
    unsigned long member2;
	unsigned long member3;
	std::vector<string> arr1;
    BCWASM_SERIALIZE(onchainData, (member1)(member2)(arr1));
};
```

对于成员内部目前只支持vector<>类型的数组的序列与反序列化，不支持c数组如char[]。

18. RETURN_CHARARRAY的定义及使用

定义

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

使用举例

```C++
const char *getName() const
{
    std::string name;
    platone::getState("NAME_KEY", name);
    size_t size = name.size();
    RETURN_CHARARRAY(name.c_str(), size + 1);
}
```

19. BCWasm合约不支持C++14的正则表达式相关函数。
20. BCWasm lib本身不支持检索区块相关信息，但是wasm虚拟机通过state.hpp对外提供了如下接口来检索常用的区块信息，比如时间戳等：

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

21. 使用ctool时指定函数参数时，参数之间需要用逗号分隔，注意中间不能有空格。
22. 目前StroageType模板的get方法是非const类型，返回的变量需要存放起来（否则会自动析构），未来需要支持cosnt类型的get方法。
23. getContractAddress(char *name, char *version), name + version version需要转为string拼接
24. wasm log日志有时打印失败，需要手动重启platone节点
25. 老版本的platone代码关于json序列化的bug已在新版本中进行了修复
26. WASM虚拟机默认只给智能合约分配了200K的内存空间，智能合约操纵的数据量总大小不可以超过这个限制。虽然可以增大虚拟机的内存空间，但是会带来虚拟机初始化速度变慢，从而导致整个区块链吞吐量的降低。以下是使用内存的主要方法：

> DB命名空间内的array，map和list等数据结构。

> bcwasm::setState()方法、bcwasm::getState()方法、malloc()方法。


