# PlatONE 权限管理教程

## 1. 系统合约简述

在PlatONE联盟链中，权限控制是由系统合约维护的，在链初始时系统上部署如下７个系统合约，用于权限管理。

| 系统合约名称 | 标识符             | 作用                     |
| :----------- | :----------------- | :----------------------- |
| 合约管理合约 | cnsManager         | 记录名称到地址的映射关系 |
| 参数管理合约 | __sys_ParamManager | 维护链的系统级参数       |
| 用户管理合约 | __sys_UserManager  | 用户管理                 |
| 用户注册合约 | __sys_UserRegister | 用户注册                 |
| 角色管理合约 | __sys_RoleManager  | 角色管理合约             |
| 角色注册合约 | __sys_RoleRegister | 角色注册合约             |
| 节点管理合约 | __sys_NodeManager  | 节点管理合约             |
|              |                    |                          |

* 系统合约支持的接口，请参照附录

ctool在`~/PlatONE-Go/cmd/SysContracts/build` ，系统合约存放在`~/PlatONE-Go/cmd/SysContracts/build/systemContract/cnsManager`下

比如可以通过调用cnsManager合约来查询各个系统合约的地址

```shell
cd ~/PlatONE-Go/cmd/SysContracts/build
./ctool invoke --addr 0x0000000000000000000000000000000000000011 --abi ./systemContract/cnsManager/cnsManager.cpp.abi.json  --config ./config.json --func getRegisteredContracts --param 0 --param 10
```

合约调用结果如下所示：

```json
{"code":0,
 "msg":"ok",
 "data":{
     "total":7,
     "contract":[
         {"name":"__sys_NodeManager","version":"1.0.0.0","address":"0xee62fbc0b45f594465738fb5c794c60f9827f59e","origin":"0x9d41fe3b45f3cacb1bebbb4e5670f88ff74ef1c4","create_time":1561359510,"enabled":true},
         {"name":"__sys_NodeRegister","version":"1.0.0.0","address":"0x6d26ea822241d1baf1c774dd46947616cd0f3d40","origin":"0x9d41fe3b45f3cacb1bebbb4e5670f88ff74ef1c4","create_time":1561359511,"enabled":true},
         {"name":"__sys_ParamManager","version":"1.0.0.0","address":"0x2f0956a40cdb7f64ecc532a9c3d7171ed193ed03","origin":"0x9d41fe3b45f3cacb1bebbb4e5670f88ff74ef1c4","create_time":1561359505,"enabled":true},
         {"name":"__sys_RoleManager","version":"1.0.0.0","address":"0xfd82ff83bbfcfc0dea1fb48f015393f4218d61a7","origin":"0x9d41fe3b45f3cacb1bebbb4e5670f88ff74ef1c4","create_time":1561359508,"enabled":true},{"name":"__sys_RoleRegister","version":"1.0.0.0","address":"0x53ba35b8b8f4b7ab63094a5e9e7bc3485a0b7bd4","origin":"0x9d41fe3b45f3cacb1bebbb4e5670f88ff74ef1c4","create_time":1561359509,"enabled":true},
         {"name":"__sys_UserManager","version":"1.0.0.0","address":"0x9ef86ec254db98b8e086e36803ce23195a66711c","origin":"0x9d41fe3b45f3cacb1bebbb4e5670f88ff74ef1c4","create_time":1561359506,"enabled":true},
         {"name":"__sys_UserRegister","version":"1.0.0.0","address":"0xae7236d1d1788612101c9ee6b25f71925db3bf0d","origin":"0x9d41fe3b45f3cacb1bebbb4e5670f88ff74ef1c4","create_time":1561359507,"enabled":true}
     ]
 }
}
```

## 2. 用户权限及管理

PlatONE中的权限都是对系统中的用户分配的，用户的角色即表示用户的权限，目前用户可以分配如下角色（权限）。

| 用户角色（权限） | 作用                                             |
| :--------------- | :----------------------------------------------- |
| chainCreator     | 链创建者，在链创建时生成，是系统中权限最高的账户 |
| chainAdmin       | 链管理员，在由链创建者设置，可以设置多个链管理员 |
| nodeAdmin        | 节点管理员，用于管理系统中的节点信息             |
| contractAdmin    | 合约管理员，可以管理系统中的合约相关的权限控制   |
| contractDeployer | 链部署者，该角色表示用户可以在链上部署合约       |

每个角色的权限范围如下表所示

|                            | 链创建者（超管） | 链管理员（普管） | 节点管理员 | 合约管理员 | 合约部署者 |
| -------------------------- | ---------------- | ---------------- | ---------- | ---------- | ---------- |
| 指定或取消链管理员         | &radic;          |                  |            |            |            |
| 指定或取消节点管理员       | &radic;          | &radic;          |            |            |            |
| 指定或取消合约管理员       | &radic;          | &radic;          |            |            |            |
| 指定或取消合约部署者       | &radic;          | &radic;          |            | &radic;    |            |
| 新加节点申请               | &radic;          | &radic;          | &radic;    |            |            |
| 管理所有的节点             | &radic;          | &radic;          |            |            |            |
| 为自己部署的合约设置防火墙 | &radic;          | &radic;          |            |            | &radic;    |
| 审核已部署的合约           | &radic;          | &radic;          |            | &radic;    |            |
| 管理自己加入的节点         | &radic;          | &radic;          | &radic;    |            |            |
| 部署合约                   | &radic;          | &radic;          |            |            | &radic;    |

链启动后，第一个部署系统合约的账号默认为链创建者（chainCreator）。

假如我们需要将一个账号A设置为链管理员(chainAdmin)，需要遵循如下步骤：

```shell
# 1. 账号A申请成为系统用户
#    使用账号A，调用用户注册合约（__sys_UserRegister）的registerUser接口
./ctool invoke                                              \
        --addr 0xae7236d1d1788612101c9ee6b25f71925db3bf0d   \
        --abi systemContract/userRegister/userRegister.cpp.abi.json                     \
        --config config.json                                \
        --func registerUser                          \
        --param '{"address":"0x33d253386582f38c66cb5819bfbdaad0910339b3","name":"userA","mobile":"13111111111","email":"userA@email.com","roles":["chainAdmin"],"remark":"平台用户申请"}'

# 2. chainCreator审核申请信息后，同意用户的申请行为
#    使用部署系统合约的账号（chainCreator）调用用户注册合约（__sys_UserRegister）的approve接口
./ctool invoke                                              \
        --addr 0xae7236d1d1788612101c9ee6b25f71925db3bf0d   \
        --abi systemContract/userRegister/userRegister.cpp.abi.json                     \
        --config config.json                                \
        --func approve                                      \
        --param "userA 的地址"                                 \
        --param 2

# 3. 查询用户信息，确认添加用户成功
#    可使用任意账号调用用户管理合约（__sys_UserRegister）的getAccountByAddress接口
./ctool invoke                                              \
        --addr 0x9d41fe3b45f3cacb1bebbb4e5670f88ff74ef1c4   \
        --abi systemContract/userRegister/userRegister.cpp.abi.json                     \
        --config config.json                                \
        --func getAccountByAddress                          \
        --param "userA 的地址"                               \
```

在PlatONE的权限中，一个用户可以有多个角色（权限）。如果需要给已存在的用户追加某个角色（权限），则根据如下步骤进行（以添加nodeAdmin和chainAdmin为例）：

```shell
# 1. chainCreator、chainAdmin这个两类角色才可以设置nodeAdmin角色，所以应该使用这两类用户调用用户注册合约（__sys_RoleManager）的addRole接口
./ctool invoke                                              \
        --addr 0xae7236d1d1788612101c9ee6b25f71925db3bf0d   \
        --abi systemContract/roleManager/roleManager.cpp.abi.json                     \
        --config config.json                                \
        --func addRole                                      \
        --param "userA"                                     \
        --param "userA 的地址"                               \
        --param '["nodeAdmin","chainAdmin"]'

# 2. 查询用户信息，确认添加用户成功
#    可使用任意账号调用用户管理合约（__sys_RoleManager）的getRolesByAddress接口
./ctool invoke                                              \
        --addr 0x9d41fe3b45f3cacb1bebbb4e5670f88ff74ef1c4   \
        --abi systemContract/roleManager/roleManager.cpp.abi.json                     \
        --config config.json                                \
        --func getRolesByAddress                             \
        --param "userA 的地址"                                     \
```

## 3. 节点准入权限

在PlatONE联盟链中，节点是通过节点管理合约来控制的。

只有chainCreator、chainAdmin和nodeAdmin这三类用户才可以设置系统合约中的节点情况，当需要添加节点、更新节点状态、删除节点时都需要这三类账号来调用合约。

### 3.1 添加观察者节点

```shell
# 1. 添加观察者节点， 观察者节点只同步区块数据，而不参与共识
#    chainCreator、chainAdmin和nodeAdmin这个三类角色才可以设置nodeAdmin角色，所以应该使用这三类用户调用用户注册合约（__sys_NodeManager）的add接口
./ctool invoke                                              \
        --addr 0xae7236d1d1788612101c9ee6b25f71925db3bf0d   \
        --abi systemContract/nodeManager/nodeManager.cpp.abi.json                     \
        --config config.json                                \
        --func add                                      \
        --param '{"name":"node-${NODE_ID}","type":${NODE_TYPE},"publicKey":"${PUBKEY}","desc":"desc","externalIP":"${IP}","internalIP":"${IP}","rpcPort":${RPC_PORT},"p2pPort":${P2P_PORT},"root":${IS_ROOT},"owner":"0x${account}","status":1}'
```

参数信息为一个json字符串，用户应该填充对应的字段，如下为一个节点完整的信息

```json
{ "name":"node's name",
  "type":0,
  "publicKey":"71bb2aa47f4ddeccf80190bf98a80136ab52d6c8e2c4d54a11e690d4dffd9578f8c0ed614039b7ee06b55c7b96fdfe9099efa5dd12c6b1115f1c4817a093c1c7",
  "desc":"node description",
  "externalIP":"208.120.201.12",
  "internalIP":"127.0.0.1",
  "rpcPort":6790,
  "p2pPort":16790,
  "root":false,
  "owner":"0xae7236d1d1788612101c9ee6b25f71925db3bf0d",
  "status":1
}
```

### 3.2 将节点升级为共识节点

```shell
# 2. 将节点升级为共识节点，共识节点即同步数据也参与共识

./ctool invoke                                              \
        --addr 0xae7236d1d1788612101c9ee6b25f71925db3bf0d   \
        --abi systemContract/nodeManager/nodeManager.cpp.abi.json                     \
        --config config.json                                \
        --func update                                      \
        --param "node name"
        --param '{"type":1}'
```

### 3.3 查询所有节点信息

```shell
# 3. 查询所有节点信息

./ctool invoke                                              \
        --addr 0xae7236d1d1788612101c9ee6b25f71925db3bf0d   \
        --abi systemContract/nodeManager/nodeManager.cpp.abi.json                     \
        --config config.json                                \
        --func getAllNodes
```

## 4. 合约部署及调用权限

### 4.1 合约部署权限

在链上部署合约的权限检查默认是不开启的，需要管理员调用参数管理合约将此功能开启才可以检查部署合约的权限。

```shell
# 1. 启用合约部署的权限检查
#    参数： 0，默认值，不检查合约部署权限； 1， 检查合约部署权限
./ctool invoke                                              \
        --addr 0xae7236d1d1788612101c9ee6b25f71925db3bf0d   \
        --abi systemContract/paramManager/paramManager.cpp.abi.json                     \
        --config config.json                                \
        --func setCheckContractDeployPermission             \
        --param 1

# 查询合约部署权限检查是否开启
./ctool invoke                                              \
        --addr 0xae7236d1d1788612101c9ee6b25f71925db3bf0d   \
        --abi systemContract/paramManager/paramManager.cpp.abi.json                     \
        --config config.json                                \
        --func getCheckContractDeployPermission             \
```

如下几种角色才可以部署合约，不具备如下角色的用户将不能在链上部署合约。

* chainCreator
* chainAdmin
* contractAdmin
* contractDeployer

假如某个账户需要部署合约，则该账户需要由具备权限的账户将其设置为`contractDeployer`

### 4.2 合约调用权限 | 合约防火墙

在PlatONE中合约的调用权限由合约防火墙设置，只有合约的创建者才可以设置对应合约的防火墙。

合约防火墙具备合约接口级别的访问控制，通过如下两个列表实现：

* ACCEPT: 可以访问相应接口的地址列表，相当于白名单
* REJECT: 拒绝访问相应接口的地址列表，相当于黑名单

```shell
# 打开指定合约的防火墙
ctool fwInvoke --addr 0x22 --func '__sys_FwOpen()' --config ./config.json

# 关闭指定合约的防火墙
ctool fwInvoke --addr 0x22 --func '__sys_FwClose()' --config ./config.json

# 设定指定合约的白/黑名单
ctool fwInvoke --addr 0x22 --func '__sys_FwAdd("ACCEPT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json

ctool fwInvoke --addr 0x22 --func '__sys_FwAdd("REJECT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json

# 清空指定合约的白/黑名单
ctool fwInvoke --addr 0x22 --func '__sys_FwClear("ACCEPT")' --config ./config.json
ctool fwInvoke --addr 0x22 --func '__sys_FwClear("REJECT")' --config ./config.json

# 删除指定合约的白/黑名单里的指定地址
ctool fwInvoke --addr 0x22 --func '__sys_FwDel("ACCEPT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json
ctool fwInvoke --addr 0x22 --func '__sys_FwDel("REJECT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json

# 重置指定合约的白/黑名单
ctool fwInvoke --addr 0x22 --func '__sys_FwSet("ACCEPT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json
ctool fwInvoke --addr 0x22 --func '__sys_FwSet("REJECT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json

# 查询指定合约的防火墙状态
ctool fwInvoke --addr 0x22 --func '__sys_FwStatus()' --config ./config.json
```

## 附录A： 系统合约接口及方法

### A-1. cnsManager

```shell
int cnsRegisterFromInit(const char * name,const char * version)

int cnsRegister(const char * name,const char * version,const char * address)

int cnsUnregister(char * name,char * version)

const char * getContractAddress(char * name,char * version)

char * getRegisteredContracts(int pageNum,int pageSize)

char * getRegisteredContractsByAddress(char * origin,int pageNum,int pageSize)

int ifRegisteredByName(char * name)

int ifRegisteredByAddress(char * address)

char * getContractInfoByAddress(char * address)

char * getHistoryContractsByName(char * name)
```

### A-2. paramManager

```shell
int setGasContractName(const char * contractName)

const char * getGasContractName()

int setCBFTTimeParam(int produceDuration,int blockInterval)

const char * getCBFTTimeParam()

int setIsProduceEmptyBlock(int isProduceEmptyBlock)

int getIsProduceEmptyBlock()

int setTxGasLimit(unsigned long long txGasLimit)

unsigned long long getTxGasLimit()

int setBlockGasLimit(unsigned long long blockGasLimit)

unsigned long long getBlockGasLimit()

int setAllowAnyAccountDeployContract(int isAllowAnyAccountDeployContract)

int setCheckContractDeployPermission(int checkPermission)

int getCheckContractDeployPermission()

int getAllowAnyAccountDeployContract()

int setIsApproveDeployedContract(int isApproveDeployedContract)

int getIsApproveDeployedContract()

int setIsTxUseGas(int isTxUseGas)

int getIsTxUseGas()
```

### A-3. userManager

```shell
int addUser(const char * userJson)

int enable(const char * userAddr)

int disable(const char * userAddr)

int delUser(const char * userAddr)

int update(const char * userAddr,const char * updateJson)

const char * getAccountByAddress(const char * address)

const char * getAccountByName(const char * name)

int isValidUser(const char * userAddr)
```

### A-4. userRegister

```shell
int registerUser(const char * registJson)

int approve(const char * userAddress,int auditStatus)

const char * getAccountByAddress(const char * address)

const char * getAccountByUsername(const char * UserName)

const char * getAccountsByStatus(int pageNum,int pageSize,int accountStatus)

int getStatusByAddress(const char * address)
```

### A-5. roleManager

```shell
int addRole(const char * name,const char * address,const char * roles)

int removeRole(const char * address,const char * roles)

const char * getRolesByAddress(const char * address)

const char * getRolesByName(const char * name)

const char * getAccountsByRole(const char * role)

int hasRole(const char * addr,const char * role)
```

### A-6. roleRegister

```shell
int registerRole(const char * roles)

int approveRole(const char * address,int status)

const char * getRegisterInfoByAddress(const char * address)

const char * getRegisterInfoByName(const char * name)

const char * getRegisterInfosByStatus(int status,int pageNum,int pageSize)
```

### A-7. nodeManager

```shell
int add(const char * nodeJsonStr)

const char * getAllNodes()

int validJoinNode(const char * publicKey)

int nodesNum(const char * nodeJsonStr)

const char * getNodes(const char * nodeJsonStr)

int update(const char * name,const char * nodeJsonStr)

const char * getEnodeNodes(int deleted)

const char * getNormalEnodeNodes()

const char * getDeletedEnodeNodes()
```
