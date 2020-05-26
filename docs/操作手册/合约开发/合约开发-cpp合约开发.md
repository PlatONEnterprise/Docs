本文档通过一系列代码示例讲解合约的各个功能，用户可以通过学习这些例子来深入理解如何编写一个应用合约。

## Contract类

Contract类是有bcwasm库提供的合约基类，用户开发的合约必须派生于该类。Contract类中定义了一个`init()`虚函数，用户合约需要实现该`init()`函数，该函数在合约首次发布时执行，仅调用一次，该方法作用类似于solidity合约中的构造函数。

注意： `init()`方法必须实现为`public`类型，这样合约在部署时才能够调用该函数来初始化合约数据。

```cpp
#include <bcwasm/bcwasm.hpp>
namespace my_namespcase {
    class my_contract : public bcwasm::Contract
    {
        public:
        my_contract(){}
        /// 实现父类: bcwasm::Contract 的虚函数
        /// 该函数在合约首次发布时执行，仅调用一次
        void init() 
        {
            /* 做一些初始化操作 */
        }
    };
}
```

## 合约外部方法

合约外部方法指的是合约可以在外部调用的接口，功能类似与solidity合约中的`public`类型方法,在bcwasm库中是通过`BCWASM_ABI`宏来定义外部方法。通过`BCWASM_ABI`声明的方法，可以在合约外部通过rpc消息调用、也可以被其他合约调用。

```cpp
#include <bcwasm/bcwasm.hpp>
namespace my_namespcase {
    class my_contract : public bcwasm::Contract
    {
        public:
        my_contract(){}
        /// 实现父类: bcwasm::Contract 的虚函数
        /// 该函数在合约首次发布时执行，仅调用一次
        void init() 
        {
            /* 做一些初始化操作 */
        }
        void setData(char * data){
            /* 修改合约数据 */
        }
    };
}
// 外部方法
BCWASM_ABI(my_namespcase::my_contract:setData)
```

## 链上存储接口

Wasm合约内置库为数据的持久化提供了`setState()`方法，可以通过调用`bcwasm::setState()`函数实现数据持久化，相应地查询调用`bcwasm::getState()`。

下方的示例合约中，有两个接口供外部调用`setData(char* data)`和`getData()`。这两个方法分别调用了`bcwasm::setState()`、`bcwasm::getState()`，以实现数据的上链持久化和查询。

```cpp
#include <bcwasm/bcwasm.hpp>
namespace my_namespcase {
    class my_contract : public bcwasm::Contract
    {
        public:
        void init(){}
        void setData(char * data){
            std::string m_data(data);
            bcwasm::setState("DataKey", m_data);
        }
        const char* getData() const{
            std::string m_data;
            bcwasm::getState("DataKey", m_data);
            return m_data.c_str();
        }
    };
}
// 外部方法
BCWASM_ABI(my_namespcase::my_contract, setData)
BCWASM_ABI(my_namespcase::my_contract, getData)
```

## Const方法

合约中的`const`类型方法提供对合约状态的只读操作，该类型声明的函数不能够修改合约数据，一般用来查询合约的链上数据。下方代码中，`getData()`即为const方法，用于查询数据。

```cpp
const char* getData() const{
    std::string m_data;
    bcwasm::getState("DataKey", m_data);
    // 读取合约数据并返回
    return m_data.c_str();
}
```

## Struct、Map

### Struct 结构体

结构体语法规则与C++一致，但是如果用于需要将结构数据上链持久化，则需要在结构体中使用`BCWASM_SERIALIZE`宏，该宏为结构体类型提供了序列化/反序列化的方式。

下方的合约示例中，定义了一个`Student_t`结构体类型，并通过合约接口`setData()`将数据持久化到链上，然后可以通过`getData()`方法查询数据。

```cpp
#include <bcwasm/bcwasm.hpp>
namespace my_namespcase {
    struct Student_t
    {
        std::string name;       // 姓名
        int64_t age;            // 年龄
        BCWASM_SERIALIZE(Student_t, (name)(age));
    };
    class my_contract : public bcwasm::Contract
    {
        public:
        void init(){}
        void setData(char * name, int64_t age){
            Student_t stu;
            stu.name = std::string (name);
            stu.age = age;
            bcwasm::setState("DataKey", stu);
        }
        const char* getData() const{
            Student_t stu;
            bcwasm::getState("DataKey", stu);
            std::stringstream ret;
            ret << "stu.name: " << stu.name << ", stu.age: " << stu.age;
            // 读取合约数据并返回
            return ret.str().c_str();
        }
    };
}
// 外部方法
BCWASM_ABI(my_namespcase::my_contract, setData)
BCWASM_ABI(my_namespcase::my_contract, getData)
```
### Map

bcwasm中提供了map类型的封装，定义map结构时，需要指定map的名称、key的类型、value的类型。

```cpp
char mapName[] = "students";
bcwasm::db::Map<mapName, std::string, Student_t> students;
```

map结构支持如下的几种api：

* `find(key)`: 根据key查找value
* `insert(key, value)`: 当map中还没有以key为索引的内容时，插入以key为索引的value
* `update(key, value)`： 当map中已经存在以key为索引的内容时，更新key对应的value

下方的示例合约中定义了一个map用于保存学生的姓名、年龄信息，以学生姓名为key作为索引，其中`setData`方法输入学生的姓名、年龄，`getData`方法根据姓名查询学生的年龄。

```cpp
#include <bcwasm/bcwasm.hpp>
namespace my_namespcase {
    struct Student_t
    {
        std::string name;       // 姓名
        int64_t age;            // 年龄
        BCWASM_SERIALIZE(Student_t, (name)(age));
    };
    // 定义一个map，保存学生姓名、年龄信息，以学生姓名为key作为索引
    char mapName[] = "students";
    bcwasm::db::Map<mapName, std::string, Student_t> students;

    class my_contract : public bcwasm::Contract
    {
        public:
        void init(){}
        void setData(char * name, int64_t age){
            Student_t stu;
            stu.name = std::string (name);
            stu.age = age;
            Student_t *stu_p = students.find(std::string(name));
            if (stu_p == nullptr){
                students.insert(stu.name, stu);
            } else{
                students.update(stu.name, stu);
            }
        }
        const char* getData(char* name) const{
            Student_t *stu = students.find(std::string(name));
            if (stu == nullptr){
                return (std::string("no such student")).c_str();
            }else{         
                std::stringstream ret;
                ret << "stu.name: " << stu->name << ", stu.age: " << stu->age;
                return ret.str().c_str();
            }
        }
    };
}
// 外部方法
BCWASM_ABI(my_namespcase::my_contract, setData)
BCWASM_ABI(my_namespcase::my_contract, getData)
```

## Event

Event允许我们方便地使用PlatONE的日志基础设施。我们可以在dapp中监听Event，当合约中产生Event时，会使相关参数被存储到交易的Log中。这些Log与地址相关联，被写入区块链中，可以通过交易Receipt查询某个交易所产生的Event。

宏`BCWASM_EVENT`和`BCWASM_EMIT_EVENT`提供了对合约Event的直接支持，使用方法如下：

```cpp
/// 定义Event.
/// BCWASM_EVENT(eventName,arguments...)
BCWASM_EVENT(setData,const char *,const int64_t)
/// 触发Event
BCWASM_EMIT_EVENT(setData,name,age);
```

我们在示例合约中加入Event事件，每次调用`setData()`时，触发Event事件，示例合约代码如下所示：

```cpp
#include <bcwasm/bcwasm.hpp>
namespace my_namespcase {
    struct Student_t
    {
        std::string name;       // 姓名
        int64_t age;            // 年龄
        BCWASM_SERIALIZE(Student_t, (name)(age));
    };
    // 定义一个map，保存学生姓名、年龄信息，以学生姓名为key作为索引
    char mapName[] = "students";
    bcwasm::db::Map<mapName, std::string, Student_t> students;

    class my_contract : public bcwasm::Contract
    {
        public:
        void init(){}
        // 定义Event
        BCWASM_EVENT(setData,const char*,int64_t)

        void setData(char * name, int64_t age){
            Student_t stu;
            stu.name = std::string(name);
            stu.age = age;
            Student_t *stu_p = students.find(std::string(name));
            if (stu_p == nullptr){
                students.insert(stu.name, stu);
            } else{
                students.update(stu.name, stu);
            }
            /// 触发Event
            BCWASM_EMIT_EVENT(setData,name,age);
        }
        const char* getData(char * name) const{
            Student_t *stu = students.find(std::string(name));
            if (stu == nullptr){
                return (std::string("no such student")).c_str();
            }else{         
                std::stringstream ret;
                ret << "stu.name: " << stu->name << ", stu.age: " << stu->age;
                return ret.str().c_str();
            }
        }
    };
}
// 外部方法
BCWASM_ABI(my_namespcase::my_contract, setData)
BCWASM_ABI(my_namespcase::my_contract, getData)
```

加入Event后我们再次调用`setData`合约，然后查询交易的Receipt，如下所示。

```java
{
  blockHash: "0xd3324a86bb4c2a9f99592ea16c02bddae6ced421c0170a07f781fb9dfa7b1d8c",
  blockNumber: 77,
  contractAddress: null,
  cumulativeGasUsed: 449872,
  from: "0x61eaf416482341e706ff048f20491cf280bc29d6",
  gasUsed: 449872,
  logs: [{
      address: "0x07894a9f9edffe4b73eb8928f76ee2993039e4d7",
      blockHash: "0xd3324a86bb4c2a9f99592ea16c02bddae6ced421c0170a07f781fb9dfa7b1d8c",
      blockNumber: 77,
      data: "0xc785676578696e1c",
      logIndex: 0,
      removed: false,
      topics: ["0xd20950ab1def1a5df286475bfce09dc88d9dcba71bab52f01965650b43a7ca8e"],
      transactionHash: "0xa4735b9dbf93f0f8d7831f893270ff8a42244141455ed308fd985b90ee9bc3f5",
      transactionIndex: 0
  }],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000800000008000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000",
  status: "0x1",
  to: "0x07894a9f9edffe4b73eb8928f76ee2993039e4d7",
  transactionHash: "0xa4735b9dbf93f0f8d7831f893270ff8a42244141455ed308fd985b90ee9bc3f5",
  transactionIndex: 0
}
```

在Receipt的logs字段中是我们通过Event产生的数据，其中主要字段的含义为：

* address: 产生该Event的合约地址
* blockHash: 产生该Event的交易所在区块哈希
* blockNumber: 产生该Event的交易所在区块号
* data: Event参数的Rlp编码，在上面合约示例中为[name,age]的RLP编码
* topics: Event名称的哈希值，在上面合约示例为字符串`"setData"`的哈希值

## 跨合约调用

bcwasm库提供了类`DeployedContract`用于跨合约调用，当需要在合约中调用其他合约时，首先使用目标合约地址初始化一个`DeployedContract`示例，然后调用对应的方法，如下所示：

```cpp
// 调用目的地址： "0x07894a9f9edffe4b73eb8928f76ee2993039e4d7"
// 调用的方法： setData(name,age)
bcwasm::DeployedContract regManagerContract("0x07894a9f9edffe4b73eb8928f76ee2993039e4d7");
char name[]= "name";
int64_t age = 18;
regManagerContract.call("setData", name, age);
```
`DeployedContract`提供如下几种调用方式：

```cpp
// 无需返回值调用
void call("funcName", arguments...);
void delegateCall("funcName", arguments...);

// string类型返回值
std::string callString("funcName", arguments...)
std::string delegateCallString("funcName", arguments...)

// Int64类型返回值
int64_t callInt64("funcName", arguments...)
int64_t delegateCallInt64("funcName", arguments...)
```

`call()`与`delegateCall()`都可以用于调用合约，但是在被调目标合约的视角来看是有区别的，使用`call()`时被调合约看到的调用者`caller`是发起该调用的合约，而当使用`delegateCall()`时，发起调用的合约直接将自身的`caller`传递给目标合约。比如在如下的两个例子中，第一种情况下，ContractB看到的`caller`是ContractA的地址；而在第二中情况中，ContractB看到的`caller`是user的地址。

```
1. user ----> ContractA --call()--> ContractB
2. user ----> ContractA --delegateCall()--> ContractB
```

## 初始化方法中注册cns合约

PlatONE在系统合约中提供了CNS服务功能，可以将合约注册至系统合约中，以实现使用合约名称版本调用合约而无需使用地址。可以在合约的初始化方法`init()`中直接将合约注册到系统合约中，以便使用CNS合约的便捷功能。

通过在`init()`方法中调用cnsManager合约的`cnsRegisterFromInit(name,version)`方法就可以实现，需要注意合约版本必须是`"x.x.x.x"`的格式。

```cpp
void init()
{
    DeployedContract reg("0x0000000000000000000000000000000000000011");
    reg.call("cnsRegisterFromInit", "name", "1.0.0.0");
}
```
## hash()

bcwasm库提供了与以太坊一致的哈希方法`sha3()`，使用方式如下所示：

```cpp
std::string  msg = "hello";
bcwasm::h256 hash = bcwasm::sha3(msg);
```

## ecrecover()

`ecrecover()`函数提供了根据原文哈希和签名恢复出签名人地址的功能，使用方法如下：

```cpp
// 针对字符串"hello"的签名
std::string  sig = "4949eb47832d8a90c8c94b57de49d11b031fcd6d6dcb18c198103d2d431e2edf07be6c3056fe054ad6d1f62a24a509426a1c76687708ab684ad609ae879399fa00";
// 签名原文
std::string  msg = "hello";
// 首先求出签名原文的哈希
bcwasm::h256 hash = bcwasm::sha3(msg);
// 通过ecrecover恢复出签名人地址
bcwasm::h160 addr = bcwasm::ecrecover(hash.data(), bcwasm::fromHex(sig).data());
```

## caller()、origin()和address()

* caller()：返回caller信息，假如合约A通过call()方法调用合约B，那么caller就是合约A的地址
* origin()：返回调用发起人的地址，不管合约间的调用情况如何，该函数始终返回最初交易发送者的地址
* address()：返回当前合约的地址

## 其他注意事项


1. 当前合约对外接口仅支持以下数据类型：
```cpp
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
void  
```

2. platone合约库对u32和定长数组bytesN未定义，目前可以分别用uint32_t和char[]数组代替。

3. 在实现合约对外接口的查询方法是时，若函数返回的是字符串（比如通过调用string的c_str()方法），需要在该函数内部新申请（malloc）一段内存，并将该字符串copy到这段新的内存，由于该内存是由BCWasm虚拟机统一管理，故不存在内存泄露问题。返回字符串类型时，可以使用`RETURN_CHARARRAY`宏实现，该宏定义如下
```cpp
#define RETURN_CHARARRAY(src，size) \
do \
{ \
    char *buf = (char *)malloc(size); \
    memset(buf，0，size); \
    strcpy(buf，src); \
    return buf; \
} \
while(0)
```

4. wasm合约内置库中的u256类型转换字符串类型需要进行如下调用：
```
u256value.convert_to<std::string>()
```