
PlatONE中的CNS服务提供了合约名称、版本到地址的映射，该功能通过系统合约`cnsManager`实现。通过CNS服务，开发者可以使用容易记忆的合约名称、版本信息调用合约，而无需合约地址。

用户部署合约后，可以将该合约注册到`cnsManager`，然后就可以通过名字及版本调用自己部署的合约。

`cnsManager`支持的API如下所示：

* 注册合约 | `cnsRegister(name, version, address)`
* 注销合约 | `cnsUnregister(name, version)`
* 查询所有已注册的合约 | `getRegisteredContracts(pageNum, pageCnt)`
* 查询合约历史版本信息 | `getHistoryContractsByName(name)`
* 查询合约地址注册信息 | `ifRegisteredByAddress(address)`
* 查询合约名称注册信息 | `ifRegisteredByName(name)`
* 查询合约名称注册信息 | `ifRegisteredByName(name)`

## 注册合约

注册合约是将用户已经部署的合约注册到`cnsManager`系统合约中，注册成功后，用户就可以通过合约名称及版本信息调用合约了。

需要注意的是，只有合约的部署者才能够将对应合约进行注册，而且合约版本号为4个通过`.`连接的数字，比如`0.0.0.0`。

示例：假设合约已经部署到链上，合约地址为`"0x2ee8d0545ebd20f9a992ff54cb0f21a153539206"`，如下操作将合约注册到`cnsManager`系统合约中，合约名称"test"，版本为"1.0.0.0"

```shell
#先切换到ctool所在目录下，{WORKSPACE}路径参考快速指南
cd ${WORKSPACE}/bin
./ctool cnsInvoke --cns  "cnsManager" --func 'cnsRegister' --param "test" --param "1.0.0.0" --param "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206"  --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

## 查询合约的注册信息

* 查询所有已注册的合约 | `getRegisteredContracts(pageNum,pageCnt)`

该方法查询所有已经注册的合约，由于注册合约数量可能较多，因此该方法支持分页查询。第一个参数`pageNum`表示查询第几页，第二个参数`pageCnt`表示每页记录的数量。

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'getRegisteredContracts' --param 0 --param 10 --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

* 查询合约历史版本信息 | `getHistoryContractsByName(name)`

该方法通过合约名称查询该名称下注册的所有版本的合约信息

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'getHistoryContractsByName' --param "test" --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```
* 查询合约地址注册信息 | `ifRegisteredByAddress(address)`

该方法查询某个合约地址是否已经被注册到了`cnsManager`系统合约中

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'ifRegisteredByAddress' --param "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206" --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

* 查询合约名称注册信息 | `ifRegisteredByName(name)`

该方法查询某个合约名称是否已经被注册了

```shell
 ./ctool cnsInvoke --cns "cnsManager"  --func 'ifRegisteredByName' --param "test" --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```
* 查询合约地址 | `getContractAddress(name,version)`

该方法通过名称及版本查询合约的地址，当版本号填`"latest"`时，表示要查询的是最高的版本的合约地址

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'getContractAddress' --param "test" --param "1.0.0.0"  --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```
## 注销合约

```shell
./ctool cnsInvoke  --cns "cnsManager"  --func 'cnsUnregister'   --param "test"   --param "1.0.0.0"  --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

## 通过CNS名称访问合约

```shell
# 假设合约已注册，名称为"contractName"，合约中包含一个方法为 Method1，接受字符串类型的入参

./ctool cnsInvoke --cns "contractName" --func "Method1"  --param "arg" --abi  <user-abi-file> --config  <user-config-file>
```
