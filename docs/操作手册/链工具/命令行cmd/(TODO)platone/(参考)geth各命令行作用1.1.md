[TOC]

# 1. Geth命令行用法

基于: go-ethereum@1.8-stable

```console
geth [选项] 命令 [命令选项] [参数…]
```

# 2. 命令

1. account    管理账户

   1. list  显示账户
   2. new 创建账户
   3. update 跟新账户
   4. import 导入账户
   5. 例子: geth account new

2. attach    :  通过rpc/ipc 连接以太坊节点

   1. 例子: geth attach http:127.0.0.1:9999

3. bug      :  浏览器打开github用于提交bug issues

   1. 例子: geth bug

4. console    启动交互式JavaScript环境

   1. 例子: geth console

5. copydb     从文件夹chaindata创建本地链

   1. 例子: geth --datadir ./peer2 copydb peer1/geth/chaindata/

6. dump      指定hash 或者 区块高度 获取此时的 所有账户 和账户对应的字段信息

   1. 例子: geth dump 0

   2. ```go
      "f76f69cee4faa0a63b30ae1e7881f4f715657010": {
                  "balance": "200000000000000000000",
                  "nonce": 0,
                  "root": "56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
                  "codeHash": "c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470",
                  "code": "",
                  "storage": {}
              },
      ...
      ```

7. dumpconfig 显示配置信息

   1. 显示 eth, Eth.Ethash, Eth.TxPool, Eth.GPO, Shh, Node, Node.P2P, Node.HTTPTimeouts, Dashboard 等配置信息
   2. 例子: geth dumpconfig

8. export     导出区块链到文件

   1. <filename> [<blockNumFirst> <blockNumLast>]
   2. 例子: `geth --datadir ./peer1 export exportfile 0 100`

9. import     导入一个区块链文件

   1. <filename> (<filename 2> ... <filename N>) 
   2. 例子: `geth --datadir ./peer2 import exportfile`

10. init       启动并初始化一个新的创世纪块

   1. 指定创世区块文件来进行初始化创世区块
   2. 例子: geth init genesis.json

11. js         执行指定的JavaScript文件(多个)

    1. <jsfile> [jsfile...]
    2. 例子: geth js xx.js

12. license    直接输入, 查看许可信息

    1. 例子:  geth license

13. makecache  生成ethash验证缓存(用于测试)

    1. <blockNum> <outputDir>
    2. 例子: geth makecache 20 xxdir

14. makedag    生成ethash 挖矿DAG(用于测试)

    1. <blockNum> <outputDir>

15. monitor    监控和可视化节点指标

    1. 需指定Available 的指标, 即可 可视化的监控
    2. 例子:   ./geth monitor txpool/underpriced/Overall

16. removedb   删除区块链和状态数据库

    1. 例子: geth --datadir peer2/ removedb

17. version    打印版本号

    1. 例子: geth version

18. wallet     管理钱包导入

    1. 例子: geth wallet import /path/to/my/presale.wallet

19. help,h     显示一个命令或帮助一个命令列表

# 3. 交易池选项

1. --txpool.nolocals
   1. locals中的账户的交易, 会在验证交易时, 进行条件豁免,  比如 locals中的低价交易是不会被剔除的.   指定nolocals 则不会对本地交易特殊对待
2. `--txpool.locals` 逗号分隔指定账户,  将会被添加到txpool的locals字段中.  
3. --txpool.journal
   1. 本地  pending和queue中的交易记录在磁盘, 方便本地节点重启
   2. 默认: "transactions.rlp"
4. -- txpool.rejournal
   1. 指定时间间隔, 重新生成交易日志.  
   2. 最小为1秒, 如果短于1秒, 会被替换成1秒
   3. 默认1hour
5. --txpool.pricelimit
   1. 加入交易池的最小的gas价格限制,  默认为1,  当设置小于1, 则会替换成1
   2. txpool对象的 gasPrice字段
6. --txpool.pricebump
   1. 每一个账号 都会被创建一个 txList,  根据交易的nonce 可以从txlist中取出对应的交易,   如果一个新的交易, 要替换老的交易 (他们的nonce 相同), 那么价格需上浮, 不然不会被加入到 txlist中,  此选项就是指定上浮百分比
   2. 默认上浮10%
7. --txpool.accountslots
   1. 每个帐户保证**可执行**的最少交易槽数量  (默认: 16)
   2. 设置小于1时,  替换成16
   3. 会进行管理一下每个本地账户的 交易状态,  如果传入nil, 那么就会把 queue中的 拿过来
   4. 在检查低nonce(太久的交易, 比现在的还低), 和 低余额超过gas的交易等等之后,  遍历pending进行均衡配额
   5. 如果penging总数大于了全局的GlobalSlots的话, 那么就会检查 每一个账户的交易,  通过策略比对 账户的 accountslots, 减少单个账户交易, 从而减少GlobalSlots总交易数
8. --txpool.globalslots
   1. 所有帐户**可执行**的最大交易槽数量, 默认: 4096
9. --txpool.accountqueue
   1. 每个帐户允许的最多**非可执行**交易槽数量 (默认: 64)
   2. 遍历每一个账户的时候, 会把账户对应的txList超过了 accountqueue 个数的交易, 进行删除掉  (根据nonce倒叙的) 
10. --txpool.globalqueue
    1. 所有帐户**非可执行**交易最大槽数量  (默认: 1024)
    2. 如果queue中的账号的总非可执行的交易数 大于指定阈值, 那么  把不是本地发起交易的账号放入数组, (进行了排序,  然后 循环,  取出最后一个账号的交易, 进行从集合中删除它的所有交易, 然后 再取出倒数第二个账号, 进行删除所有交易, 直到 非可执行交易的总量在 阈值之内
11. --txpool.lifetime
    1. 非可执行交易最大入队时间(默认: 3小时)
    2. txpool的queue集合中, 如果一个非本地账户 存在的时间超过了 txpool.lifttime指定的时间, 那么将会删除此账户的全部交易 (每分钟检查一次)

# 4. ethereum选项

1. --config value          
   1. TOML 配置文件
2. --datadir “xxx”        
   1.  数据库和keystore密钥的数据目录
3. --keystore              
   1. keystore存放目录(默认在datadir内)
4. --nousb                 
   1. 禁用监控和管理USB硬件钱包
5. --networkid value       
   1. 网络标识符(整型, 1=Frontier, 2=Morden (弃用), 3=Ropsten, 4=Rinkeby) (默认: 1)
6. --testnet               
   1. Ropsten网络:预先配置的POW(proof-of-work)测试网络
7. --rinkeby               
   1. Rinkeby网络: 预先配置的POA(proof-of-authority)测试网络
8. --syncmode "fast"       
   1. 同步模式 ("fast", "full", or "light")
9. --ethstats value        
   1. 统计上报ethstats service  URL (nodename:secret@host:port) 
10. --identity value        
    1. 自定义节点名
11. --lightserv value       
    1. 指定大于0, 则会增加les服务,  根据比例设置线程数
12. --lightpeers value      
    1. 最大的LES客户端的peers数量(默认值:100)
13. --lightkdf   
    1. 启动时, 会在 账号配置的时候, 降低加密使用的内存和cpu资源

# 5. 开发者选项

1. --dev               使用POA共识网络，

   1. 默认预分配一个开发者账

   2. 自动开启挖矿

   3. 无法使用p2p网络

2. --dev.period value  开发者模式下挖矿周期 (0 = 仅在交易时) (默认: 0)

# 6. 性能调优选项

1. --trie-cache-gens   
   1. 保持在内存中产生的trie node数量(默认:120)
   2. stateDB在 New和 reset的时候,  都会调用 OpenTrie,  此时会给 trie的cachelimit设置为指定的数量
2. --cache
   1. 设置可供使用的内存数, 默认是 1024M .  cache.database, cache.trie, cache.gc等内存 都是指定百分比, 从cache上面进行分配

# 7. 账户选项

1. --unlock 
   1. 需解锁账户以逗号分隔
2. --password
   1. 用于非交互式密码输入密码文件,   指定文件路径,  每一行作为 一个账号对应的密码

# 8. 网络选项

1. --bootnodes value    用于P2P发现引导的enode urls(逗号分隔)(对于light servers用v4+v5代替)
   1. 当开启了节点发现,  满足 节点连接数为0, 20秒内没有连接过节点等条件时, 将会尝试连接bootnodes所指定的节点
2. --bootnodesv4 , --bootnodesv5 设置后, 会覆盖掉 `--bootnodes`设置的内容
3. --port value         p2p的监听端口(默认值:30303)
4. --maxpeers value    p2p.server的 maxpeers 最大连接数 (如果设置为0，网络将被禁用)(默认值:25)
   1. 节点建立连接时, 会判断此时是否超过了最大节点数,  如果是, 则不会再建立连接
5. --maxpendpeers value    最大尝试连接的数量(如果设置为0，则将使用默认值)(默认值:0)
   1. 被动接受 网络中最大的尝试连接数, 如果设置为0(默认值), 都将会在p2p.server的 listenLoop中, 被按照50来执行
6. --nat value             NAT端口映射机制 (any|none|upnp|pmp|extip:<IP>) (默认: “any”)
   1. 把内网的ip+端口, 映射为  路由器的IP+端口。
   2. 默认值为any, 为同时使用 upnp和pmp
7. --nodiscover            禁用节点发现机制(手动添加节点)
   1. 关闭节点发现,  同步模式为light 则默认关闭.  同时p2p中不会尝试连接bootnodes中节点
8. --v5disc           启动新的节点发现机制: topic-discovery协议
   1. 当启动 `--v5disc`, 那么 会启动节点发现机制, `nodiscover`设置为true也不管用. 
9. --nodekey value    
   1. 指定p2p节点私钥文件路径
10. --nodekeyhex value     指定私钥十六进制字符串  进行启动
    1. nodekey和nodekeyhex不能同时设置

# 9. 旷工选项

1. --mine             启动挖矿 
   1. 当设置启动 `--dev`时, 也会在节点启动时启动挖矿(如果, 此时同时指定了`--sync light`则会报错)
2. `--minerthreads`/`--miner.threads`:      挖矿使用的CPU线程数量,   默认为0
   1. 如果设置为0, 则在启动挖矿时, 被当做-1来进行使用, 则会禁止本地挖矿
3. --etherbase / `--miner.etherbas`       挖矿奖励地址(默认=第一个创建的帐户)(默认值:“0”) 
4. --targetgaslimit / `--miner.gastarge`  目标gas限制：设置最低gas限制(默认值:“8000000”)
   1. 设置到 worker的 gasFloor字段,  
   2. 会根据 gasFloor 和 gasCeil 来进行计算 下一个区块的区块头中的 gasLimit, 进行一个公式的调整
5. `--miner.gaslimit`  挖矿是, 区块的gas上限, 默认8000000
   1. 设置到 worker的 gasCeil上
6. --gasprice / `--miner.gasprice`        挖矿接受交易的最低gas价格 默认GWei
   1. 设置在 eth的 gasPrice对象上, 在挖矿时, 会设置到txpool的gasprice, 也就是限制接收交易的gasprice
7. --extradata / `--miner.extradat`        矿工设置的额外块数据 (不设置, 那么默认填客户端版本)
   1. 长度不能超过32
   2. 提交新区块时, 会填到 header的Extra字段中
8. `--miner.recommit` 新的挖掘循环的时间间隔 默认3秒
   1. 如果设置小于一秒, 则会被当做一秒使用
   2. 在newWorkLoop的时候, 会指定 一个 时间间隔传入 ,  会根据 默认的调整比例 进行调整
9. `--miner.noverify`  不对远程区块进行验证, ethash中, 不验证接收到的区块的工作量

# 10. GAS价格选项

1. --gpoblocks value      用于检查gas价格的最近块的个数  (默认: 20)
   1. 设置小于1, 将被当做1使用
   2. 检查最近指定个数的区块的交易的gasprice
2. --gpopercentile value  建议gas价参考最近交易的gas价的百分位数，(默认: 50)
   1. 设置小于0 被当做0, 设置大于100 被当做100
   2. 会把指定的最近的个数的区块 的所有交易的 gasPrice 进行排序之后,  然后以指定的百分比, 从集合中找出对应的 gasprice为建议的price

# 11. 虚拟机选项

1. --vmdebug        记录VM和合约调试信息 bool

   1. 也就是在opSha3的时候, 日志记录了对应的hash值

      

# 12. 日志调试选项

1. --metrics            
   1. 启用metrics收集和报告bool
   2. 开启之后, 每3秒收集一次运行时进程的各项指标,  正在使用的内存之类的等等
2. --fakepow            不真正的进行挖矿和验证区块
3. --verbosity value    
   1. 设置log等级
   2. 0=crit, 1=error, 2=warn, 3=info, 4=debug, 5=detail (default: 3)
4. --vmodule value      
   1. 不同的模块中, 可以设置不同的日志等级.  (比如 eth/*=6,p2p=5)
5. --backtrace value    
   1. 指定文件和对应行号, 把这行的log的调用栈都打印出来 (比如 “block.go:271”)
6. --debug       日志会带上此log执行所在的文件名和对应行号
7. --pprof     会启用pprof HTTP的服务器, 可以看到程序 开启了多少个goroutine, 多少个线程等, 用于内存分析
8. --pprofaddr value           pprof HTTP服务器监听接口(默认值:127.0.0.1)
9. --pprofport value           pprof HTTP服务器监听端口(默认值:6060)
10. --memprofilerate value      按指定频率进行内存分析    默认512 * 1024
   1. 用于pprof,  按照指定的字节数 进行每次采样, 采样率越小, 数据越细致, 但进程运行越慢
11. --blockprofilerate value    按指定频率打开block profiling    (默认值:0)
    1. 获取导致阻塞的 goroutine 堆栈,   指定的数值的意思是, 如指定为1,则每发生1个goroutine堵塞, 就会采样报告一次. 设置为0 , 则此采样操作取消
12. --cpuprofile          指定文件路径,写入cpu的报告分析, 显示每一个函数执行的时间
13. --trace     将runtime中trace写入指定文件, 帮助追踪os线程如何调度goroutine ,  goroutine为什么堵塞等等问题

# 13. whisper实验选项

1. --shh                       
   1.  启用Whisper 安全通信协议 
   2. `--dev`情况下, 默认开启. 
   3. 只要设置了whisper相关的命令行参数, 都会自动开启 whisper
2. --shh.maxmessagesize   可接受的最大的消息大小 (默认值: 1024*1024)
   1. 设置和接收消息时, 会检查消息的大小, 超过阈值, 会显示错误
3. --shh.pow value              
   1. 可接受的最小的POW (默认值: 0.2)
   2. 接收到的请求信息, 会比对 powtarget值, 在 指定值之上 才会被接收
   3. 相当于计算pow的成本,  为了防止垃圾消息, 也是为了缓解网络压力  (pow值也 消息大小和TTL成正比)
4. --shh.restrict-light  限制 2个轻客户端之间的连接 
   1. 在握手中, 进行判断,  如果是轻节点 在 加上此参数设置后, 那么 节点间握手将不会被通过

# 14. ethash选项

1. --ethash.cachedir                        ethash验证缓存目录(默认 = datadir目录内)
2. --ethash.cachesinmem value               在内存保存的最近的ethash缓存个数  (每个缓存16MB ) (默认: 2)
3. --ethash.cachesondisk value              在磁盘保存的最近的ethash缓存个数 (每个缓存16MB) (默认: 3)
4. --ethash.dagdir ""                       存ethash DAGs目录 (默认 = 用户hom目录)
5. --ethash.dagsinmem value                 在内存保存的最近的ethash DAGs 个数 (每个1GB以上) (默认: 1)
6. --ethash.dagsondisk value                在磁盘保存的最近的ethash DAGs 个数 (每个1GB以上) (默认: 2)

# 15. API和控制台选项

1. --rpc
   1. 启用HTTP-RPC服务器
2. --rpcaddr value
   1. HTTP-RPC服务器接口地址(默认值:“localhost”)
3. --rpcport value
   1. HTTP-RPC服务器监听端口(默认值:8545)
4. --rpcapi value
   1. 基于HTTP-RPC接口提供的API
5. --ws
   1. 启用WS-RPC服务器
6. --wsaddr value
   1. WS-RPC服务器监听接口地址(默认值:“localhost”)
7. --wsport value 
   1. WS-RPC服务器监听端口(默认值:8546)
8. --wsapi  value
   1. 基于WS-RPC的接口提供的API
9. --wsorigins value
   1. websockets请求允许的源
10. --ipcdisable
    1. 禁用IPC-RPC服务器
11. --ipcpath
    1. 包含在datadir里的IPC socket/pipe文件名(转义过的显式路径)
12. --rpccorsdomain value
    1. 允许跨域请求的域名列表(逗号分隔)(浏览器强制)
13. --jspath loadScript
    1. JavaScript加载脚本的根路径(默认值:“.”)
14. --exec value 
    1. 执行JavaScript语句(只能结合console/attach使用)
    2. 例子:  geth --exec "eth.blockNumber" console 2>/dev/null
15. --preload value
    1. 预加载到控制台的JavaScript文件列表(逗号分隔)

# 16. 补充选项

1. `--goerli`
   1.  预先配置的POA测试网络
2. `--override.constantinople` 
   1. 君士坦丁堡分岔设置
3. `--docroot` 
   1. 文档根目录, 默认home
4. `--gcmode`  区块链垃圾回收模式 ("full", "archive") 默认full
   1. 链退出时, full 会将 最近区块账户状态写入磁盘, archive不会
   2. archive不会对trie进行修剪, 会实时同步tire到磁盘, full不会
5. `--whitelist` (<number>=<hash>) 使用逗号分隔
   1. 发送GetBlockHeadersMsg消息请求白名单中对应的区块头
   2. 收到对应的区块后, 会 验证此区块hash是否跟白名单中hash 匹配得上
6. `--dashboard` 
   1. 启动dashboard, 收集以太坊节点数据，用于可视化分析
7. `--dashboard.addr`   
   1.  dashboard监听的host  默认localhost
8. `--dashboard.host`    
   1. dashboard监听的port 默认8080
9. `--dashboard.refresh`    
   1. 刷新间隔, 默认5s
10. `--vm.ewasm` 外部的ewasm配置
    1. ./geth --vm.ewasm="$HOME/testnet/libhera.so,metering=true,fallback=true" ...
11. `--vm.evm`  外部evm配置
12. `import-preimages`  hash预镜像导出到RLP编码的流
    1.  geth --datadir ./data0 export-preimages exportPreImage
13. `export-preimages`   从RLP编码的流导入hash预镜像
    1.  geth --datadir ./data1 import-preimages exportPreImage

