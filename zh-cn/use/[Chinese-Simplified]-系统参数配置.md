
### 是否出空块

PlatONE默认配置是不出空块，当用于需要出空块时，可以通过系统合约更改配置。

查询是否出空块， 1表示出空块，0表示不出空块
```
#先切换到ctool所在目录下,{WORKSPACE}路径参考快速指南
cd ${WORKSPACE}/bin
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func getIsProduceEmptyBlock
```

设置为出空块

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setIsProduceEmptyBlock --param 1
```

设置为不出空块

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setIsProduceEmptyBlock --param 0
```

### 区块gaslimit

PlatONE采用区块gaslimit限制一个区块中所有交易的最大运算量，该值是通过系统合约管理的，可以通过如下的方式查询管理。

查询当前区块gaslimit

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func getBlockGasLimit
```

设置新的区块gaslimit

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setBlockGasLimit --param 20000000000
```

### 交易gasLimit
PlatONE采用交易gaslimit限制一个交易的最大运算量，该值是通过系统合约管理的，可以通过如下的方式查询管理。

查询当前交易gaslimit

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func getTxGasLimit
```

设置新的交易gaslimit

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setTxGasLimit --param 2000000000
```

### 是否检查合约部署权限
PlatONE支持对合约部署权限进行检查，具体权限范围参见权限模型章节

查询是否检查合约部署权限

```
# 0表示不检查，任意用于可以部署合约，1表示检查只有特定用户才可以部署合约
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func getCheckContractDeployPermission
```

启用合约部署权限检查

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setCheckContractDeployPermission --param 1
```
关闭合约部署权限检查

```
./ctool  cnsInvoke --cns __sys_ParamManager --abi ../conf/contracts/paramManager.cpp.abi.json --config ../conf/ctool.json --func setCheckContractDeployPermission --param 0
```


