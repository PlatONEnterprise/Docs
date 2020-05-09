
### Whether to generate empty blocks

Empty blocks are not generated in the default configuration. When needed to generate empty blocks, the configuration can be changed through the system contract.

Query whether generate empty blocks, 1 means yes, 0 means no.

```shell
# Switch to the directory where ctool is located, please reference the quick guide to find  {WORKSPACE} path.
cd ${WORKSPACE}/bin
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func getIsProduceEmptyBlock
```

Switch to generate empty blocks
```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setIsProduceEmptyBlock --param 1
```

Switch to not generate empty blocks

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setIsProduceEmptyBlock --param 0
```

### Block gaslimit

PlatONE uses the block gaslimit to limit the maximum amount of calculations for all transactions in a block. This value is managed by the system contract and can be queried by the following methods.

Query the current block gaslimit

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func getBlockGasLimit
```

Set new block gaslimit

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setBlockGasLimit --param 20000000000
```

### Transaction gaslimit
PlatONE  limits the maximum amount of calculations for a transaction by the transaction gaslimit . This value is managed by the system contract and can be queried and managed as follows.

Query current transaction gaslimit

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func getTxGasLimit
```

Set new transaction gaslimit

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setTxGasLimit --param 2000000000
```
### Whether  to check contract deployment permissions
You can check  the contract deployment permissions in PlatONE. For details, see the permissions model section.

Query whether to check contract deployment permissions

```
# 0 means  arbitrarily used to deploy the contract without check, 1 means it need to be checked and only certain users can deploy the contract
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func getCheckContractDeployPermission
```

Check contract deployment permission 

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setCheckContractDeployPermission --param 1
```
Stop contract deployment permission check

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setCheckContractDeployPermission --param 0
```
