+ 防火墙开启
+ 防火墙配置
+ 防火墙状态查询

开启防火墙

```shell
ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwOpen()' --config ../config.json 
```



更新防火墙数据

```shell
ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwAdd("Accept","*:getName")' --config ../config.json 

ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwAdd("Accept","*:setName")' --config ../config.json 
```

```shell
 ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwAdd("Reject","0xab91e9382c9e095efe03bb23e3bbea4b5c92f830:setName")' --config ../config.json 
```

说明：防火墙过滤规则样式：(”{地址}：{函数}“)。



清除防火墙数据

```shell
 ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwClear("Reject")' --config ../config.json 
```



防火墙状态查询

```shell
 ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwStatus()' --config ../config.json 
```

防火墙关闭

```shell
ctool fwInvoke --addr "0x5855455df9d1b395332e3ae9b2efc4e18ad8755f" --func '__sys_FwClose()' --config ../config.json
```


