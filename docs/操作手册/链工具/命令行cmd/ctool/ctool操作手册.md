# ctool操作手册

## 1. 配置文件 config.json

ctool中的部分参数会从配置文件中读取，若未指定`-config`，默认情况下从当前目录读取`config.json`文件。

config.json文件展示：

```json
{
  "url":"http://192.168.9.73:6789",
  "gas": "0x76c0",
  "gasPrice": "0x9184e72a000",
  "from":"0xfb8c2fa47e84fbde43c97a0859557a36a5fb285b"
}
```

## 2. 合约部署 | ctool deploy

```bash
./ctool deploy
-abi        abi json file path (must)
-code       wasm file path (must)
-config     config path  (optional)

e.g： ./ctool deploy -abi "D:\\resource\\temp\\contractc.cpp.abi.json" -code "D:\\resource\\temp\\contractc.wasm"
```

## 3. 合约调用

### 3.1 普通调用 | ctool invoke

```bash
./ctool invoke
-addr     contract address (must)
-func     functon name and param (must)
-abi      abi json file path (must)
-type     transaction type ,default 2 (optional)

e.g: ./ctool invoke -addr "0xFC43e7f481b9d3F75CcfFc8D23eAC522E96dE570" -func "transfer("a",b,c) " -abi "D:\\resource\\temp\\contractc.cpp.abi.json" -type
```

### 3.2 合约名称调用 | ctool cnsInvoke

```bash
./ctool cnsInvoke
-cns      contract name (must)
-func     functon name and param (must)
-abi      abi json file path (must)
-type     transaction type ,default 2 (optional)
-config   config path  (optional)

e.g: ./ctool cnsInvoke --cns "test" -func "transfer("a",b,c) " -abi "D:\\resource\\temp\\contractc.cpp.abi.json"
```

## 4. 转账 | ctool sendTransaction

```bash
./ctool sendTransaction
-from       msg sender (must)
-to         msg acceptor (must)
-value      transfer value (must)
-config     config path (optional)

e.g: ./ctool transfer --from "0xb239401ecf8427f17c6de134d6a6bddd3100251f" --to "0X123" --value "10"
```

## 5. 交易回执查询 | ctool getTxReceipt

```bash
./ctool getTxReceipt
-hash       txhash (must)
-config     config path (optional)

e.g: ./ctool getTxReceipt -hash <transaction_hash> -config "../config.json"
```

## 6. 防火墙操作 | ctool fwInvoke

```bash
./ctool fwInvoke
-addr     contract address (must)
-func     functon name and param (must)
-config   config path (optional)

# 开启防火墙
e.g: ./ctool fwInvoke --addr "0xacda4dfbbd6d093cf7e348abb33296d9aeb0f23c" --func '__sys_FwOpen()' --config "../config.json"
```
