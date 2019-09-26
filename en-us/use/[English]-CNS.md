1. Register contract to cns

```shell
 ctool invoke --addr "0x0000000000000000000000000000000000000011" --func 'cnsRegister' --param "test" --param "1.0.0.0" --param "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206"  --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/cnsManager.cpp.abi.json --config ../config.json 
```

2. Get register information on cns
+ list registered contracts

```shell
ctool invoke --addr "0x0000000000000000000000000000000000000011" --func 'getRegisteredContracts' --param 0 --param 10 --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/cnsManager.cpp.abi.json --config ../config.json
```
+ get contract history

```shell
ctool invoke --addr "0x0000000000000000000000000000000000000011" --func 'getHistoryContractsByName' --param "test" --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/cnsManager.cpp.abi.json --config ../config.json 
```
+ query contract register info

```shell
 ctool invoke --addr "0x0000000000000000000000000000000000000011" --func 'ifRegisteredByAddress' --param "0x2ee8d0545ebd20f9a992ff54cb0f21a153539206" --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/cnsManager.cpp.abi.json --config ../config.json
```

```shell
 ctool invoke --addr "0x0000000000000000000000000000000000000011" --func 'ifRegisteredByName' --param "test" --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/cnsManager.cpp.abi.json --config ../config.json
```
+ get contract address by name

```shell
ctool invoke --addr "0x0000000000000000000000000000000000000011" --func 'getContractAddress' --param "test" --param "1.0.0.0" --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/cnsManager.cpp.abi.json --config ../config.json
```

3. unregister contract

```shell
ctool invoke --addr "0x0000000000000000000000000000000000000011" --func 'cnsUnregister' --param "test"  --param "1.0.0.0" --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/cnsManager.cpp.abi.json --config ../config.json 
```


