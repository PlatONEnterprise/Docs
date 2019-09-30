PlatONE智能合约体系完全不同于传统智能合约，主要特点：

- 位于PlatONE的分布式服务平台
- 面向计算，FaaS (Functions as a Service)
- 底层虚拟机BCWasm
- 合约开发语言C++
- 合约内置库（PlatONE lib）



## 1. 配置Wasm合约开发环境

获取PlatONE项目源码并编译

```
# 获取PlatONE源码
git clone --recursive https://github.com/PlatONEnterprise/PlatONE-Go.git
export WORKSPACE=${PWD}/PlatONE-Go/release/linux
# 编译PlatONE
cd PlatONE-Go; make all; cd ..
# 如果编译失败. 请确保你安装了依赖环境中全部软件. 然后重新尝试.
```

定义合约工程路径：
```
cd PlatONE-Go
export CONTRCATSSPACE=${PWD}/cmd/SysContracts
```
在`${CONTRCATSSPACE}`目录下包含了合约编译所需的材料，合约源码目录为`${CONTRCATSSPACE}/appContract`

获取PlatONE最新稳定版的发布包，并进行解压。进入解压获得的PlatONE_linux目录，然后定义路径：
```
wget https://github.com/PlatONEnterprise/PlatONE-Go/releases/download/v0.9.0/PlatONE_linux_v0.9.0.tar.gz

tar -xzvf PlatONE_linux_v0.9.0.tar.gz

cd PlatONE_linux
export BINSPACE=${PWD}/bin
```

## 2. 构建项目及编译合约

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

    /// 实现父类: bcwasm::Contract 的虚函数
    /// 该函数在合约首次发布时执行，仅调用一次
    void init() 
    {
        bcwasm::println("init success...");
    }
    /// 定义Event.
    /// BCWASM_EVENT(eventName, arguments...)
    BCWASM_EVENT(setName, const char *)
    
    public:
    void setName(const char *msg)
    {
        // 定义状态变量
        bcwasm::setState("NAME_KEY", std::string(msg));
        // 日志输出
        // 事件返回
        BCWASM_EMIT_EVENT(setName, "std::string(msg)");
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
BCWASM_ABI(demo::my_contract, setName)
BCWASM_ABI(demo::my_contract, getName)
```

### 2.2. 编译合约

进入`build`目录并编译该合约

```
cd ${CONTRCATSSPACE}/build
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

其中，发布合约时需要用到的文件为my_contract.wast和my_contract.cpp.abi.json。

## 3. 部署及调用合约

### 3.1. 生成合约部署账户

```
cd ${CONTRCATSSPACE}/build
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

配置文件字段简述：

- `url` PlatONE节点开放的JSON-RPC地址信息；
- `from` 发布合约者的账户地址。

长时间没用使用账户发送过交易，或者节点重启后，账户会被锁定，不能发送交易。需要使用如下命令，重新为账户解锁。

```
../script/unlock_account.sh \
             --account 0x08e7988e60ab5aa49d1f7aa9435ac91b9fcf772c \
            --phrase "your_phrase" \
            --ip 127.0.0.1 \
            --rpc_port 6791
```

### 3.2. 部署合约

```
cd ${CONTRCATSSPACE}/build

# ctool是用来部署合约及发送交易的工具。
cp  ${BINSPACE}/ctool  .

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
- 提示2： 发布合约到PlatONE网络，需要连接到节点，并保证发布合约的账户已进行了解锁操作，且没有超时。执行git-bash.exe打开git-bash窗口。
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

BCWasm为wasm合约开发工具，其对应路径为`${CONTRCATSSPACE}`

### 5.1. BCWasm合约编码规范：

1. 自定义合约必须继承bcwasm::Contract类，并实现init()方法。合约必须实现父合约的init虚函数，该函数在合约首次发布时执行，仅调用一次，该方法作用类似于solidity合约中的构造函数。
```
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
}
```
2. 合约对外接口需要通过BCWASM_ABI宏导出，比如BCWASM_ABI(demo::FirstDemo, getName)。
```
BCWASM_ABI(demo::my_contract, getName)
```

3. 合约对外查询接口，该接口必须声明为其所在合约类的const型成员函数，否则编译没有错误但是于外部调用该合约接口是返回数据为空。
```
const char* getName() const 
{
    std::string value;
    bcwasm::getState("NAME_KEY", value);
    // 读取合约数据并返回
    return value.c_str();
}
```
4. Wasm合约内置库为数据的持久化提供了setState()方法，可以通过调用bcwasm::setState()函数实现数据持久化，相应地查询调用bcwasm::getState()。

```
// 持久化
bcwasm::setState("NAME_KEY", std::string(msg));

// 查询
std::string value;
bcwasm::getState("NAME_KEY", value);
```

5. 当前合约对外接口仅支持以下数据类型：

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

6. 对自定义数据结构进行序列化需要在结构体声明中加入BCWASM_SERIALIZE宏，具体用法如下：

```C++
// 定义了Transaction结构，其中包含三个成员，结构序列化时会将三个成员全部序列化
struct Transaction {  
	Address destination;  
	Address from;  
	int64_t value;  
	BCWASM_SERIALIZE(Transaction, (destination)(from)(value))  
};
```
7. platone合约库对u32和定长数组bytesN未定义，目前可以分别用uint32_t和char[]数组代替。

8. 在实现合约对外接口的查询方法是时，若函数返回的是字符串（比如通过调用string的c_str()方法），需要在该函数内部新申请（malloc）一段内存，并将该字符串copy到这段新的内存，由于该内存是由BCWasm虚拟机统一管理，故不存在内存泄露问题。返回字符串类型时，可以使用`RETURN_CHARARRAY`宏实现，该宏定义如下

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
9. wasm合约内置库中的u256类型转换字符串类型需要进行如下调用：

```
u256value.convert_to<std::string>()
```

10. 宏BCWASM_EVENT和BCWASM_EMIT_EVENT提供了对合约event的直接支持。

```
/// 定义Event.
/// BCWASM_EVENT(eventName, arguments...)
BCWASM_EVENT(setName, const char *)

/// 触发Event
BCWASM_EMIT_EVENT(setName, "std::string(msg)");
```