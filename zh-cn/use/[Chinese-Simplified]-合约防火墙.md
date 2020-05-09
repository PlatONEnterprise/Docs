## 合约防火墙API

PlatONE通过合约防火墙可以控制合约的访问权限，只有合约的部署者可以设置相应合约的防火墙参数。合约防火墙通过黑白名单控制访问者权限，其中黑名单优先级高于白名单。在PlatONE中合约防火墙默认是关闭的，需要合约部署者开启防火墙后才可以控制访问权限。

防火墙过滤规则样式：`"address:method"`，当需要添加多条防火墙规则时，请使用`|`分割多条规则，比如`"address1:method1|address2:method2"`。如果需要允许某个用户访问合约的所有方法，那么可以在规则中使用`*`表示适配所有方法，比如`"address:*"`；同样如果允许所有用户访问某个方法，可以使用`*:method`。

防火墙参数可以通过如下API进行设置：

+ 开启防火墙 | `__sys_FwOpen()`
+ 关闭防火墙 | `__sys_FwClose()`
+ 防火墙状态查询 | `__sys_FwStatus()`
+ 添加防火墙规则 | `__sys_FwAdd("Accept/Reject"，"address:method")`
    + 添加白名单 | `__sys_FwAdd("Accept"，"address:method")`
    + 添加黑名单 | `__sys_FwAdd("Reject"，"address:method"`
+ 删除防火墙规则 | `__sys_FwDel("Accept/Reject"，"address:method")`
    + 删除白名单 | `__sys_FwDel("Accept"，"address:method")`
    + 删除黑名单 | `__sys_FwDel("Reject"，"address:method")`
+ 重置防火墙数据 | `__sys_FwSet("Accept/Reject"，"address1:method1|address2:method2")`
    + 重置白名单 | `__sys_FwSet("Accept"，"address1:method1|address2:method2")`
    + 重置黑名单 | `__sys_FwSet("Reject"，"address1:method1|address2:method2")`
+ 防火墙数据清除 | `__sys_FwClear("Accept/Reject")`
    + 清除白名单 | `__sys_FwClear("Accept")`
    + 清除黑名单 | `__sys_FwClear("Reject")`

## 开启/关闭防火墙 

开启合约防火墙

```shell
./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwOpen()' --config ../config.json 
```

关闭合约防火墙

```shell
./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwClose()' --config ../config.json
```

## 防火墙状态查询

```shell
 ./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwStatus()' --config ../config.json 
```

## 添加防火墙规则

```shell
# 允许任意地址的账户访问合约的getName方法，以及地址为0xabcd的账户访问setName
./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwAdd("Accept"，"*:getName|0xabcd:setName")' --config ../config.json 
# 允许地址为0xabcd的账户访问合约的setName方法
./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwAdd("Accept"，"0xabcd:setName")' --config ../config.json 
```

```shell
# 拒绝地址为0xabcd的账户访问合约的setName方法
 ./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwAdd("Reject"，"0xabcd:setName")' --config ../config.json 
```

## 重置防火墙规则

```shell
# 重置指定合约的白/黑名单
ctool fwInvoke --addr 0x22 --func '__sys_FwSet("ACCEPT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json
ctool fwInvoke --addr 0x22 --func '__sys_FwSet("REJECT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json
```

## 删除防火墙规则

```shell
# 删除指定合约的白/黑名单里的指定地址
ctool fwInvoke --addr 0x22 --func '__sys_FwDel("ACCEPT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json
ctool fwInvoke --addr 0x22 --func '__sys_FwDel("REJECT", "0x33:*|*:funcName2|0x55:funcName3")' --config ./config.json
```

## 清除防火墙数据

```shell
# 清除防火墙黑名单
./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwClear("Reject")' --config ../config.json 
```
