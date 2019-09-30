1. 注册合约

```shell
#先切换到ctool所在目录下,{WORKSPACE}路径参考快速指南
cd ${WORKSPACE}/bin
# 假设合约已经部署到链上，合约地址为"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206"
# 本操作将合约注册到CNS系统合约中，合约名称"test"，版本为"1.0.0.0"
# cnsRegister("test", "1.0.0.0", "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206")
./ctool cnsInvoke --cns  "cnsManager" --func 'cnsRegister' --param "test" --param "1.0.0.0" --param "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206"  --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

2. 查询合约的注册信息

查询所有已注册的合约

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'getRegisteredContracts' --param 0 --param 10 --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

查询合约历史版本信息

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'getHistoryContractsByName' --param "test" --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```
查询合约注册信息

```shell
./ ctool cnsInvoke --cns "cnsManager"  --func 'ifRegisteredByAddress' --param "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206" --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

```shell
 ./ctool cnsInvoke --cns "cnsManager"  --func 'ifRegisteredByName' --param "test" --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```
查询合约地址

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'getContractAddress' --param "test" --param "1.0.0.0"  --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

3. 通过CNS名称访问合约

```shell
# 假设合约已注册， 名称为"contractName"， 合约中包含一个方法为 Method1，接受字符串类型的入参

./ctool cnsInvoke --cns "contractName" --func "Method1"  --param "arg" --abi <user-abi-file> --config  <user-config-file>
```

4. 注销合约

```shell
./ctool cnsInvoke  --cns "cnsManager"  --func 'cnsUnregister'   --param "test"   --param "1.0.0.0"  --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```
