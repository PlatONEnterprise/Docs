# Java - SDK

PlatONE Java SDK是面向java开发者，提供的PlatONE联盟链的java开发工具包，提供了在应用层（java 代码）访问区块链节点并获取服务的接口，比如部署合约、调用合约、查询链上数据等。

## 1. 下载与安装

请首先下载SDK最新版本的发布包，[下载地址](https://github.com/PlatONEnterprise/PlatONE-Go/releases)。

将发布包解压到本地目录，如下所示：

```shell
# 下载
wget https://github.com/PlatONEnterprise/PlatONE-Go/releases/download/v0.9.0/java_sdk_linux_v0.9.0.tar.gz
# 解压
tar -zxvf java_sdk_linux_v0.9.0.tar.gz && export SDKPATH=java-sdk
```

安装依赖环境

```shell
# java版本：jdk1.8
sudo apt install cmake g++ maven
# 安装maven依赖
cd ${SDKPATH}/bin && ./mvn.sh
```

创建java项目并在maven配置文件中添加如下的依赖项：

```js
    <dependencies>
        <dependency>
            <groupId>com.platone.client</groupId>
            <artifactId>core</artifactId>
            <version>0.4.1</version>
        </dependency>
        <dependency>
            <groupId>com.platone.client</groupId>
            <artifactId>crypto</artifactId>
            <version>0.4.1</version>
        </dependency>
        <dependency>
            <groupId>com.platone.client</groupId>
            <artifactId>abi</artifactId>
            <version>0.4.1</version>
        </dependency>
        <dependency>
            <groupId>com.platone.client</groupId>
            <artifactId>rlp</artifactId>
            <version>0.4.1</version>
        </dependency>
        <dependency>
            <groupId>com.platone.client</groupId>
            <artifactId>tuples</artifactId>
            <version>0.4.1</version>
        </dependency>
        <dependency>
            <groupId>com.platone.client</groupId>
            <artifactId>utils</artifactId>
            <version>0.4.1</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.5</version>
        </dependency>
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>3.8.1</version>
        </dependency>
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>logging-interceptor</artifactId>
            <version>3.8.1</version>
        </dependency>
        <dependency>
            <groupId>io.reactivex</groupId>
            <artifactId>rxjava</artifactId>
            <version>1.2.4</version>
        </dependency>
        <dependency>
            <groupId>org.java-websocket</groupId>
            <artifactId>Java-WebSocket</artifactId>
            <version>1.3.8</version>
        </dependency>
        <dependency>
            <groupId>com.github.jnr</groupId>
            <artifactId>jnr-unixsocket</artifactId>
            <version>0.15</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.8.5</version>
        </dependency>
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcprov-jdk15on</artifactId>
            <version>1.54</version>
        </dependency>
    </dependencies>
```

## 2. 连接节点

首先需要与PlatONE节点建立连接，以获取链上有关服务。PlatONE支持建立http连接和websocket连接两种方式。

```java
//http短连接
Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));
```

```java
//ws长连接
WebSocketClient webSocketClient = new WebSocketClient(newURI("ws://127.0.0.1:6791"));
WebSocketService ws = new WebSocketService(webSocketClient，true);
ws.connect();
Web3j web3j = Web3j.build(ws);
```

说明：

1. 建立Websocket连接需要显式调用connect方法（与HTTP不同）。
2. PlatONE节点需要在启动时打开websocket监听功能，即启动时加入参数：--ws。

## 3. 合约交互

为了方便在java项目中调用链上合约，需要首先生成合约对应的java类，在项目中创建合约类实例后，便可以调用合约。

### 3.1 合约骨架生成

1. 编写合约(以demo为例)，编写合约的步骤请参阅[Wasm合约开发](../操作手册/合约开发/合约开发-cpp合约开发.md)。

    ```cpp

        #include <stdlib.h>
        #include <string.h>
        #include <string>
        #include <bcwasm/bcwasm.hpp>

        namespace demo {
            class FirstDemo : public bcwasm::Contract
            {
                public:
                    FirstDemo(){}

                    /// 实现父类: bcwasm::Contract 的虚函数
                    /// 该函数在合约首次发布时执行，仅调用一次
                    void init()
                    {
                        bcwasm::println("init success...");
                    }
                public:
                    void setName(const char *msg)
                    {
                        // 定义状态变量
                        bcwasm::setState("NAME_KEY"，std::string(msg));
                    }

                    const char* getName() const
                    {
                        std::string value;
                        bcwasm::getState("NAME_KEY"，value);
                        // 读取合约数据并返回
                        return value.c_str();
                    }
            };
        }
        // 此处定义的函数会生成ABI文件供外部调用
        BCWASM_ABI(demo::FirstDemo，setName)
        BCWASM_ABI(demo::FirstDemo，getName)
    ```

    合约编译后会产生demo.cpp.abi.json和demo.wasm，在生成java合约代码时需要用到这两个文件。

2. 使用合约骨架生成工具生成java合约骨架:

    ``` shell

        cd java_sdk_linux_v0.9.0/java-sdk/bin
        ./client-sdk wasm generate --javaTypes      \
                </path/to/demo.wasm>                \
                </path/to/demo.cpp.abi.json>        \
                -o </path/to/src/main/java>         \
                -p <com.your.organisation.name>     \
                -t wasm
    ```

   说明：把尖括号内的内容替换成自己的内容。
   运行后会生成合约对应的java类。
   java类中包含了合约中的方法，方便在应用层中调用合约。

### 3.2 合约操作

1. 部署合约

    ```java

        //optional
        class NodeConfiguration {
                public static final String WALLETSOURCE = "/home/username/Work/PlatONE/data/keystore/keyfile.json";
                public static final String DEMOBIN = "/home/user/Work/client-sdk-0.4.1/contract/firstdemo.wasm";
            }

        //建立连接
        Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));

        //加载钱包
        Credentials credentials = WalletUtils.loadCredentials("<wallet password>"，NodeConfiguration.WALLETSOURCE);

        //部署合约  
        byte[] dataBytes = Files.readBytes(new File(NodeConfiguration.DEMOBIN));
        String binData = Hex.toHexString(dataBytes);
        Firstdemo demo = Firstdemo.deploy(web3j，credentials，binData，new DefaultWasmGasProvider()).send();
    ```

2. 加载合约

    ```java

        //optional
        class NodeConfiguration {
                public static final String WALLETSOURCE = "/home/username/Work/PlatONE/data/keystore/keyfile.json";
                public static final String DEMOBIN = "/home/user/Work/client-sdk-0.4.0/contract/firstdemo.wasm";
            }

        //建立连接
        Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));

        //加载钱包
        Credentials credentials = WalletUtils.loadCredentials("<wallet password>"，NodeConfiguration.WALLETSOURCE);

        //加载合约
        byte[] dataBytes = Files.readBytes(new File(NodeConfiguration.DEMOBIN));
        String binData = Hex.toHexString(dataBytes);
        Firstdemo contract = Firstdemo.load(binData，“<contract address>”，web3j，credentials，new DefaultWasmGasProvider());
    ```

3. 调用合约示例

    在合约部署后，客户端可以通过合约地址进行合约调用。

    1. 合约地址

        ```java

            public  static void main(String args[]) {

                Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));

                try {
                    // 密钥账户，keyfile.json为ethkey工具生成的账户文件，参照《PlatONE密钥工具文档》
                    Credentials credentials = WalletUtils.loadCredentials("1"，"/home/wxuser/keyfile.json");

                    // 合约数据
                    byte[] dataBytes = Files.readBytes(new File("/home/user/PlatONE-Workspace-0.2/contracts/build/appContract/demo/demo.wasm"));
                    String binData = Hex.toHexString(dataBytes);

                    // 加载合约
                    Demo demo = Demo.load(binData，"0x1d7f2695b43be56f52f24baa199420f8c10ac1d3"，web3j，credentials，new DefaultWasmGasProvider());

                    // 调用demo合约的setName方法，参数输入字符串"platone"
                    TransactionReceipt ret = demo.setName("platone").send();
                    System.out.println("Transaction Hash: "+ret.getTransactionHash());

                    // 调用demo合约的getName方法
                    System.out.println("getName: " +  demo.getName().send());

                }catch (Exception e){
                    System.out.println(e);
                }
            }
        ```

    2. 合约名称

        ```java

            public static void main(String[] args) {
                try {
                    Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));
                    Credentials credentials = WalletUtils.loadCredentials("1"，"/home/wxuser/keyfile.json");
                    byte[] dataBytes = Files.readBytes(new File("/home/user/PlatONE-Workspace-0.2/contracts/build/appContract/demo/demo.wasm"));
                    String binData = Hex.toHexString(dataBytes);
                    // load contract
                    CnsManager cns = CnsManager.load(null，"0x0000000000000000000000000000000000000011"，web3j，credentials，new DefaultWasmGasProvider());
                    TransactionReceipt r = cns.cnsRegister("demo"，"1.0.0.0"，"0x1d7f2695b43be56f52f24baa199420f8c10ac1d3").send();
                    if (r.isStatusOK()){
                        Demo d = Demo.load(null，"demo"，web3j，c，new DefaultWasmGasProvider());
                        d.setName("cns").send();
                        System.out.println(d.getName().send());
                        }

                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("Done...");
                }
            }
        ```

### 3.3 订阅事件

1. 订阅区块:

   在新区块产生时，client可以得到节点的区块数据推送。

   ```java

        Subscription sub = web3j.blockObservable(false).subscribe( block -> {
            System.out.println(block.getBlock().getNumber());
        });
   ```

2. 订阅event:

   在合约中可以自定义事件，client通过订阅事件的方式来获悉合约调用中所触发的事件。

   合约中定义如下的event，每次setName被调用时，就会触发该event。

   ```c++

    // event定义
    BCWASM_EVENT(setName，const char *)

    void setName(const char *msg)
    {
        // 定义状态变量
        bcwasm::setState("NAME_KEY"，std::string(msg));
        // 日志输出
        // 事件返回
        BCWASM_EMIT_EVENT(setName，"std::string(msg)");
    }
   ```

   ```java

    String contractAddress = "0x1d7f2695b43be56f52f24baa199420f8c10ac1d3";
    String eventHash = Hash.sha3String("setName");

    EthFilter filter = new EthFilter(DefaultBlockParameterName.EARLIEST，DefaultBlockParameterName.LATEST，contractAddress).addSingleTopic(eventHash);

    Subscription subTx = web3j.ethLogObservable(filter).subscribe(log -> {
        System.out.println("output: " + log.getData());
    }
   ```

   说明：Filter实例化的输入，第三个是合约的地址，第四个是Topic的哈希值（SHA-3），返回结果中log的Data字段是事件值的rlp编码。

### 3.4 web3 api调用

```java
web3j.ethBlockNumber(); // 当前最新区块高度
web3j.ethGetTransactionByHash("0x..."); // 根据交易哈希多去交易内容
web3j.ethGetTransactionReceipt("0x..."); // 根据交易哈希获取交易的回执
```
