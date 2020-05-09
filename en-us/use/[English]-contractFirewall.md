+ activate firewall
+ firewall management
+ query firewall status

Activate firewall:

```shell
./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwOpen()' --config ../config.json 
```

Update firewall data:

```shell
./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwAdd("Accept","*:getName")' --config ../config.json 

./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwAdd("Accept","*:setName")' --config ../config.json 
```

```shell
 ./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwAdd("Reject","0xab91e9382c9e095efe03bb23e3bbea4b5c92f830:setName")' --config ../config.json 
```

ps: filter rule format(”{addr}：{func}“)

Clear firewall data:

```shell
./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwClear("Reject")' --config ../config.json 
```

Query firewall status:

```shell
 ./ctool fwInvoke --addr "0x48d3a2dc59cfc2d7349f1aaf01b1b2f15cc7a611" --func '__sys_FwStatus()' --config ../config.json 
```

Close the firewall:

```shell
./ctool fwInvoke --addr "0x5855455df9d1b395332e3ae9b2efc4e18ad8755f" --func '__sys_FwClose()' --config ../config.json
```