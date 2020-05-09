1. Register contract to cns

```shell
# first switch to the directory where ctool is located, please reference the quick guide to find  {WORKSPACE} path.
cd ${WORKSPACE}/bin
# Suppose the contract has been deployed to the chain and the contract address is "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206"
# This operation registers the contract into   CNS  with the contract name "test" and the version  "1.0.0.0".
# cnsRegister("test", "1.0.0.0", "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206")
./ctool cnsInvoke --cns  "cnsManager" --func 'cnsRegister' --param "test" --param "1.0.0.0" --param "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206"  --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

2. Get registered information on cns
+ list registered contracts

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'getRegisteredContracts' --param 0 --param 10 --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```
+ get contract history

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'getHistoryContractsByName' --param "test" --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```
+ query contract register info

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'ifRegisteredByAddress' --param "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206" --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

```shell
 ./ctool cnsInvoke --cns "cnsManager"  --func 'ifRegisteredByName' --param "test" --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```
+ get contract address by name

```shell
./ctool cnsInvoke --cns "cnsManager"  --func 'getContractAddress' --param "test" --param "1.0.0.0"  --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```

3.get the contract through the CNS name

```shell
# Suppose the contract is registered, the name is "contractName", and the contract contains a method "Method1", accepting the input of the string type.

./ctool cnsInvoke --cns "contractName" --func "Method1"  --param "arg" --abi  <user-abi-file> --config  <user-config-file>
```

4. unregister contract

```shell
./ctool cnsInvoke  --cns "cnsManager"  --func 'cnsUnregister'   --param "test"   --param "1.0.0.0"  --abi  ../conf/contracts/cnsManager.cpp.abi.json  --config  <user-config-file>
```


