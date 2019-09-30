## Overview

PlatONE Java SDK is a development toolset for  PlatONE permissioned chain. 

It provides interfaces for Java developers to attach blockchain nodes and acquire related service in application layer, such as  deploying contract, invoking contract functions, search blockchain data, etc.



## Getting started

+ Install or Import

 1. Requirement: jdk1.8

 2. Maven configuration

    ```shell
    # install depended tools
    sudo apt install cmake g++ maven
    
    # install maven dependency 
    cd PlatONE-Workspace/java-sdk/bin
    ./mvn.sh 
    ```

    modify maven configuration file: add depended projects:

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

+ Establish Web3 Connnection

  In order to get blockchain services, client needs to establish connection with PlatONE nodes. 

  We support to establish the connection with `http` or  `websocket` protocols.

  ```java
  //http connection
  Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));
  ```

  ```java
  //websocket connection
  WebSocketClient webSocketClient = new WebSocketClient(newURI("ws://127.0.0.1:6791"));
  WebSocketService ws = new WebSocketService(webSocketClient,true);
  ws.connect();
  Web3j web3j = Web3j.build(ws);
  ```
  
  PS:
  
- Establishing a websocket connection requires explicitly calling the connect method (unlike HTTP)
  - The PlatONE node needs to open the websocket listening port at startup, that is, add parameters when starting: --ws

   

## Contract

+ Generate java class for contract instance

 1. You could implement a wasm contract instance, such as `demo`.  For more details of writing wasm contract, please refer to  `PlatONE_Contract_Deployment_Tutorial.md`

    contract example:
    
    ```c++
    #include <stdlib.h>
    #include <string.h>
    #include <string>
    #include <bcwasm/bcwasm.hpp>
    
    namespace demo {
        class FirstDemo : public bcwasm::Contract
        {
            public:
                FirstDemo(){}
                void init() 
                {
                    bcwasm::println("init success...");
                }
    
    
            public:
                void setName(const char *msg)
                {    
                    bcwasm::setState("NAME_KEY", std::string(msg));
                }
    
                const char* getName() const 
                {
                    std::string value;
                    bcwasm::getState("NAME_KEY", value);
                    return value.c_str();
                }
        };
    }
    
    BCWASM_ABI(demo::FirstDemo, setName)
    BCWASM_ABI(demo::FirstDemo, getName)
    ```
    
    The `demo.cpp.abi.json` and `demo.wasm`  will be generated after compiling contract. These two files will be used to generate java contract code.
    
    
    
2. We could run the following command to generate java class of the contract：

   ```shell
   cd java_sdk_linux_v0.9.0/java-sdk/bin
   ./client-sdk wasm generate --javaTypes      \
           </path/to/demo.wasm>                \
           </path/to/demo.cpp.abi.json>        \
           -o </path/to/src/main/java>         \
           -p <com.your.organisation.name>     \
           -t wasm
   ```
   
   
   Then it will generate the java class of the contract. `Java class` includes contracts' methods, which makes it convenient  to invoke contract in java application clients.



+ Contract operation

1. deploy contract

   ```java
     //optional    
     class NodeConfiguration {
             public static final String WALLETSOURCE = "/home/username/Work/PlatONE/data/keystore/keyfile.json";
             public static final String DEMOBIN = "/home/user/Work/client-sdk-0.4.0/contract/firstdemo.wasm";
         }
   
     //establish connection
     Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));
   
     //load wallet
     Credentials credentials = WalletUtils.loadCredentials("<wallet password>", NodeConfiguration.WALLETSOURCE);
                 
     //deploy contract
     byte[] dataBytes = Files.readBytes(new File(NodeConfiguration.DEMOBIN));
     String binData = Hex.toHexString(dataBytes);
     Firstdemo demo = Firstdemo.deploy(web3j, credentials, binData, new DefaultWasmGasProvider()).send();
   ```

2. load contract

   ```java
       //optional    
     class NodeConfiguration {
             public static final String WALLETSOURCE = "/home/username/Work/PlatONE/data/keystore/keyfile.json";
             public static final String DEMOBIN = "/home/user/Work/client-sdk-0.4.0/contract/firstdemo.wasm";
         }
   
     //establish connection
     Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));
   
    //load wallet 
     Credentials credentials = WalletUtils.loadCredentials("<wallet password>", NodeConfiguration.WALLETSOURCE);
     
     //load existing contract
     byte[] dataBytes = Files.readBytes(new File(NodeConfiguration.DEMOBIN));
     String binData = Hex.toHexString(dataBytes);
     Firstdemo contract = Firstdemo.load(binData, “<contract address>”, web3j, credentials, new DefaultWasmGasProvider());
   ```

   

3. call contract functions

   Application client could invoke contract functions by contract address or contract name through CNS service.
   
   Examples:
   
   1. contract address:
   
   ```java
   public  static void main(String args[]) {
       try {
           Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));
           Credentials credentials = WalletUtils.loadCredentials("1", "/home/wxuser/keyfile.json");
   
           byte[] dataBytes = Files.readBytes(new File("/home/user/PlatONE-Workspace-0.2/contracts/build/appContract/demo/demo.wasm"));
           String binData = Hex.toHexString(dataBytes);
   
           // load contract
           Demo demo = Demo.load(binData,"0x1d7f2695b43be56f52f24baa199420f8c10ac1d3", web3j, credentials, new DefaultWasmGasProvider());
   
           // invoke setName method of demo contract, the input is "platone"
           TransactionReceipt ret = demo.setName("platone").send();
           System.out.println("Transaction Hash: "+ret.getTransactionHash());
   
           // invoke getName method of demo contract
           System.out.println("getName: " +  demo.getName().send());
   
       }catch (Exception e){
        System.out.println(e);
    }
   }
   ```
   
   2. contract name
   
```java
   public static void main(String[] args) {
   	try {
   		Web3j web3j = Web3j.build(new HttpService("http://127.0.0.1:6791"));
           Credentials credentials = WalletUtils.loadCredentials("1", "/home/wxuser/keyfile.json");
           byte[] dataBytes = Files.readBytes(new File("/home/user/PlatONE-Workspace-0.2/contracts/build/appContract/demo/demo.wasm"));
           String binData = Hex.toHexString(dataBytes);
           // load contract
           CnsManager cns = CnsManager.load(null, "0x0000000000000000000000000000000000000011", web3j, credentials, new DefaultWasmGasProvider());
   		TransactionReceipt r = cns.cnsRegister("demo", "1.0.0.0", "0x1d7f2695b43be56f52f24baa199420f8c10ac1d3").send();
   		if (r.isStatusOK()){
   			Demo d = Demo.load(null, "demo", web3j, c, new DefaultWasmGasProvider());
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




   + Subscribe Event

     We support subscribe for block and self-defined topic.

     Block filters provide notification of the creation of new transactions or blocks on the network.

     Topic filters are more flexible. These allow you to create a filter based on specific criteria that you provide.

1. Subscribe Blocks:

   ```java
   Subscription sub = web3j.blockObservable(false).subscribe( block -> {          
       System.out.println(block.getBlock().getNumber());
   });
   ```


2. Subscribe Event:

   We could define event in the contract, and then client could obtain notification of the event once the event is triggered by contract call.
   For example, we define an event in the following contract, when `setName` is invoked, then a response event will be triggered.
   
   ```C++
   // event definition
   BCWASM_EVENT(setName, const char *)
   
   void setName(const char *msg)
   {
       bcwasm::setState("NAME_KEY", std::string(msg));
       BCWASM_EMIT_EVENT(setName, "std::string(msg)");
   }
   ```
   
   ```java
   //client filter
   String contractAddress = "0x1d7f2695b43be56f52f24baa199420f8c10ac1d3";
   String eventHash = Hash.sha3String("setName");
   
   EthFilter filter = new EthFilter(DefaultBlockParameterName.EARLIEST, DefaultBlockParameterName.LATEST,contractAddress).addSingleTopic(eventHash);
   
   Subscription subTx = web3j.ethLogObservable(filter).subscribe(log -> {
       System.out.println("output: " + log.getData());
   }
   ```
   
   **Note**：As for filter instantiation input, the third is the contract address, the fourth is the topic hash value (SHA-3), the return data field of the log is the event value with RLP encoded.
   	

+ web3 api:

```java
web3j.ethBlockNumber(); // current block height
web3j.ethGetTransactionByHash("0x..."); // get transaction by transaction hash
web3j.ethGetTransactionReceipt("0x..."); // get the receipt by transaction hash
```

