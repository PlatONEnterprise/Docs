## 用户角色管理

+ 用户注册
+ 用户信息查询
+ 用户管理
+ 角色管理

用户发起注册申请

```shell
ctool cnsInvoke --cns "__sys_UserRegister" --func 'registerUser({"address":"0xdc755eec2b1621fea466cc022e7d11707cb1b487","name":"Alice","mobile":"13111111111","email":"alice@wx.bc.com","roles":["chainAdmin"],"remark":"平台用户申请"})' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/userRegister.cpp.abi.json --config ../config.json
```

管理员审核用户的注册申请

```shell
ctool cnsInvoke --cns "__sys_UserRegister" --func 'approve("0xdc755eec2b1621fea466cc022e7d11707cb1b487",2)' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/userRegister.cpp.abi.json --config  ../admin.json 
```



用户信息查询

```shell
ctool cnsInvoke --cns "__sys_UserManager" --func 'getAccountByName("Alice")' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/userManager.cpp.abi.json --config ../config.json  
```

```shell
ctool cnsInvoke --cns "__sys_UserManager"  --func 'isValidUser("0xdc755eec2b1621fea466cc022e7d11707cb1b487")' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/userManager.cpp.abi.json --config ../config.json 
```



用户信息更新

```shell
ctool cnsInvoke --cns "__sys_UserManager" --func 'update("0xdc755eec2b1621fea466cc022e7d11707cb1b487",{"address":"0xdc755eec2b1621fea466cc022e7d11707cb1b487","name":"Alice","mobile":"1312222","email":"123@qq.com","status":0})' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/userManager.cpp.abi.json --config ../config.json 
```



删除用户

```shell
ctool cnsInvoke --cns "__sys_UserManager" --func 'delUser("0xb239401ecf8427f17c6de134d6a6bddd3100251f")' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/userManager.cpp.abi.json --config ../config.json
```



用户角色申请

```shell
ctool cnsInvoke --cns "__sys_RoleRegister" --func 'registerRole("["nodeAdmin"]")' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/roleRegister.cpp.abi.json --config ../config.json
```

管理员审核用户的角色申请

```shell
ctool cnsInvoke --cns "__sys_RoleRegister" --func 'approveRole("0xdc755eec2b1621fea466cc022e7d11707cb1b487",2)'  --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/roleRegister.cpp.abi.json --config ../admin.json 
```



查询用户的角色信息

```shell
ctool cnsInvoke --cns "__sys_RoleManager" --func 'getRolesByName("Alice")' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/roleManager.cpp.abi.json --config ../config.json 
```

```shell
ctool cnsInvoke --cns "__sys_RoleManager" --func 'getAccountsByRole("chainAdmin")' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/roleManager.cpp.abi.json --config ../config.json
```



移除用户角色

```shell
ctool cnsInvoke --cns "__sys_RoleManager" --func 'removeRole("0xdc755eec2b1621fea466cc022e7d11707cb1b487","["chainAdmin"]")' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/roleManager.cpp.abi.json --config ../admin.json
```



## 节点管理

+ 节点注册
+ 节点信息查询
+ 节点管理

新节点发起注册申请

```shell
ctool cnsInvoke --cns "__sys_NodeRegister" --func 'registerNode' --param '{"name":"nodeA","desc":"i am nodeA","type":0,"publicKey":"7caae651e633769fa693f633e3a468775bf19699ccdc014eb9449c2eb0f82ebde4a7d28c9b846d2df02d1e7f7e1460fbb31eb83c6faf208dcbfd4f2944d31c61","externalIP":"127.0.0.1","internalIP":"127.0.0.1","rpcPort":6793,"p2pPort":16793,"root":false}' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/contracts/nodeRegister.cpp.abi.json --config ../config.json
```

节点管理员审核节点的注册申请

```shell
ctool cnsInvoke --cns "__sys_NodeRegister" --func 'approve' --param "7caae651e633769fa693f633e3a468775bf19699ccdc014eb9449c2eb0f82ebde4a7d28c9b846d2df02d1e7f7e1460fbb31eb83c6faf208dcbfd4f2944d31c61" --param 1 --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/nodeRegister.cpp.abi.json --config ../config.json
```


查询节点信息

```shell
ctool cnsInvoke --cns "__sys_NodeManager" --func 'getNodes({"name":"node-0"})' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/nodeManager.cpp.abi.json --config ../config.json
```

```shell
ctool cnsInvoke --cns "__sys_NodeManager" --func 'getAllNodes()' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/nodeManager.cpp.abi.json --config ../config.json
```

更新节点信息

```shell
ctool cnsInvoke --cns "__sys_NodeManager" --func 'update("node-4","1")' --abi /home/wxuser/temp/PlatONE-Workspace/chain/PlatONE_linux/conf/contracts/nodeManager.cpp.abi.json --config ../config.json 
```

说明：将节点从观察者节点升级成共识节点


