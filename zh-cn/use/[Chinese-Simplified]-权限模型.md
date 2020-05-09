## 用户角色管理

+ 用户注册
+ 用户信息查询
+ 用户管理
+ 角色管理

用户发起注册申请

```shell
./ctool cnsInvoke --cns "__sys_UserRegister" --func 'registerUser({"address":"0xdc755eec2b1621fea466cc022e7d11707cb1b487"，"name":"Alice"，"mobile":"13111111111"，"email":"alice@wx.bc.com"，"roles":["chainAdmin"]，"remark":"平台用户申请"})'  --abi  ../conf/contracts/userRegister.cpp.abi.json  --config   <user-config-file>
```

管理员审核用户的注册申请

```shell
./ctool cnsInvoke --cns "__sys_UserRegister" --func 'approve("0xdc755eec2b1621fea466cc022e7d11707cb1b487"，2)' --abi ../conf/contracts/userRegister.cpp.abi.json --config  <admin-config-file>
```



用户信息查询

```shell
./ctool cnsInvoke --cns "__sys_UserRegister" --func 'getAccountByUsername("Alice")'  --abi  ../conf/contracts/userRegister.cpp.abi.json  --config   <user-config-file>
```

```shell
./ctool cnsInvoke --cns "__sys_UserManager"  --func 'getAccountByAddres("0xdc755eec2b1621fea466cc022e7d11707cb1b487")'  --abi  ../conf/contracts/userManager.cpp.abi.json   --config   <user-config-file>
```



用户信息更新

```shell
#用户只能更新mobile及email，status由管理员通过enable，disable，delUser更改
./ctool cnsInvoke --cns "__sys_UserManager" --func 'update("0xdc755eec2b1621fea466cc022e7d11707cb1b487"，{"address":"0xdc755eec2b1621fea466cc022e7d11707cb1b487"，"name":"Alice"，"mobile":"1312222"，"email":"123@qq.com"，"status":0})' --abi  ../conf/contracts/userRegister.cpp.abi.json   --config   <user-config-file>
```



设置用户可用，禁用、删除用户

```shell
./ctool cnsInvoke --cns "__sys_UserManager" --func 'enable("0xb239401ecf8427f17c6de134d6a6bddd3100251f")'  --abi ../conf/contracts/userManager.cpp.abi.json --config  <admin-config-file>
```

```shell
./ctool cnsInvoke --cns "__sys_UserManager" --func 'disable("0xb239401ecf8427f17c6de134d6a6bddd3100251f")'   --abi ../conf/contracts/userManager.cpp.abi.json --config  <admin-config-file>
```

```shell
./ctool cnsInvoke --cns "__sys_UserManager" --func 'delUser("0xb239401ecf8427f17c6de134d6a6bddd3100251f")'   --abi ../conf/contracts/userManager.cpp.abi.json --config  <admin-config-file>
```



用户角色申请

```shell
./ctool cnsInvoke --cns "__sys_RoleRegister" --func 'registerRole("["nodeAdmin"]")' --abi ../conf/contracts/roleRegister.cpp.abi.json --config <user-config-file>
```

管理员审核用户的角色申请

```shell
./ctool cnsInvoke --cns "__sys_RoleRegister" --func 'approveRole("0xdc755eec2b1621fea466cc022e7d11707cb1b487"，2)'  --abi ../conf/contracts/roleRegister.cpp.abi.json --config <admin-config-file>
```



查询用户的角色信息

```shell
./ctool cnsInvoke --cns "__sys_RoleManager" --func 'getRolesByName("Alice")' --abi ../conf/contracts/roleManager.cpp.abi.json --config <user-config-file>
```

```shell
./ctool cnsInvoke --cns "__sys_RoleManager" --func 'getAccountsByRole("chainAdmin")' --abi ../conf/contracts/roleManager.cpp.abi.json --config <user-config-file>
```




## 节点管理

+ 添加节点
+ 节点管理

添加节点信息

```shell
./ctool cnsInvoke --cns "__sys_NodeManager" --func 'add' --param ' {"name":"node1"，"owner":"0x4FCd6fe35f0612C7866943cb66C1d93eb0746bcC"，"approveor": "0xdF6518e51e0d90A3CBa26e4AdFE74454E2D90BdE"，"desc": "i love this world"，"type":1，"publicKey":"0x81ec63a2335c1f79244cbe485eb6bffef48cfd7df58b1009411c6114670eefd27da865914c70f7e49ceeb1002f1c24f4930975a2eb05cb5ac1373bed83a9932a"，"externalIP": "127.0.0.1"，"internalIP": "127.0.0.1"，"rpcPort": 6789，"p2pPort": 16789，"status": 1}'  --abi  ../conf/contracts/nodeManager.cpp.abi.json --config <user-config-file>
```

节点管理员更新节点信息

```shell
./ctool cnsInvoke --cns "__sys_NodeManager" --func 'update' --param '{"type":0}'   --abi  ../conf/contracts/nodeManager.cpp.abi.json --config   <admin-config-file>
```


查询节点信息

```shell
./ctool cnsInvoke --cns "__sys_NodeManager" --func 'getNodes({"name":"node1"})' --abi  ../conf/contracts/nodeManager.cpp.abi.json   --config   <user-config-file>
```

```shell
./ctool cnsInvoke --cns "__sys_NodeManager" --func 'getAllNodes()' --abi  ../conf/contracts/nodeManager.cpp.abi.json   --config   <user-config-file>
```



