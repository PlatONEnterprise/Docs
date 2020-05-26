# IBFT 共识算法

## 1. 概述

PlatONE 中的共识为高度优化的BFT类共识算法，其容错率为1/3，在保留即时确认（instant finality）的关键特性的同时,极大地提高了去中心化的程度。共识可以保证上链的区块是确定的，也就是说链不会出现分叉，同时每一个有效的区块都会插入到链上。

PlatONE 的共识支持超过100个共识节点。相对于其他一些常见的BFT共识，PlatONE 的共识的性能有显著的提升。在10个共识节点的情况下，TPS 接近 1000。

PlatONE 的共识运行的相关参数可以灵活地进行配置，并且 PlatONE 的共识中的共识节点集合可以灵活地进行更新。近期计划支持共识的插件化，以及共识的可审计性等。

PlatONE 共识是在 round 上进行的。在特定的 round 上，通过预先设置的策略选取一个出块者节点。出块者节点的选取策略目前支持两种：round robin 和 sticky proposer。

出块者节点提议区块后，各共识节点进行共识。共识分三阶段，其中后两个阶段为投票阶段，用以保证 Safety。PlatONE 共识使用 round change 机制结合锁定和解锁机制来保证共识的的 liveness 。通过优化解锁机制，解决了业界多个知名项目内存在的共识死锁问题。

PlatONE 共识会为每一个链上的区块生成共识证明，也就是对于该区块的各共识节点的有效签名，因而区块可以进行自验证，同时也能支持轻节点。

区块中如果不包含交易，则称为空区块。PlatONE 目前支持不出空区块，也就是上链的区块中都含有交易。不出空区块的机制可以有效地节省区块链占用的存储空间。

以下具体介绍 PlatONE 中的共识算法。

## 2. 共识节点选取机制

* 节点的类型和状态

  节点分为共识节点（validator）和观察员节点两种类型。对于共识节点来说，存在两种状态：正常和隔离。只有处于正常状态的共识节点才可以参与共识和打包区块。

* 共识节点的选取机制
  
  节点管理（NodeManager）系统合约设计用于存储和管理节点信息。可以通过节点申请（NodeRegister）系统合约申请注册共识节点，审核通过后，申请节点的类型会更新为共识节点，更新后的节点信息存储在节点管理合约中，并且可被查询。
  
  管理员可以根据需要更新共识节点的状态，来决定共识节点是否可以参加共识。
  
* 共识节点集合的获取
  
  链上每次产生新区块后，节点管理合约中最新的节点信息都会被读取，并且最新的共识节点集合会被保存下来，并被共识引擎读取和使用。

## 3. 共识流程

### 3.1. 正常流程

#### 3.1.1. 定义

  以下是一些重要术语或概念的定义。

  * `+2/3` 表示"超过 2/3".
  * `NEW ROUND`:  新的round中会确定一个新的区块提议者（比如采用round robin算法），在新的round开始时，各共识节点等待接收`PRE-PREPARE`消息。
  * `PRE-PREPARED`:  validator节点接收到了`PRE-PREPARE` 消息，同时广播`PREPARE`消息之后进入这种状态。之后，validator节点等待并接收`+2/3`的`PREPARE` 或 `COMMIT` 消息。（注：有的validator节点因锁定在提议区块上，会在收到`PRE-PREPARE` 消息后直接广播`COMMIT` 消息。因此，这里validator节点等待并接收`PREPARE` 或 `COMMIT` 消息）
  * `PREPARED`: validator节点接收到了`+2/3`的`PREPARE`消息，同时广播`COMMIT`消息之后进入这种状态。之后，validator节点等待并接收`+2/3`的 `COMMIT` 消息。
  * `COMMITTED`:  validator节点接收到了`+2/3`的`COMMIT` 消息，进入到这种状态。此时，可以将提议的区块插入到区块链上了
  * `FINAL COMMITTED`:  新的区块成功上链后，validator节点进入到这种状态。此时，节点准备进入下一个round
  * `ROUND CHANGE`:  validator节点等待接收`+2/3`的、针对同一个提议round的`ROUND CHANGE`消息

#### 3.1.2. 选取proposer的规则

  + Round robin 算法（目前采用的）

  + Sticky proposer

#### 3.1.3. 共识流程（三阶段协议）

  共识流程由三个阶段组成：`PRE-PREPARE`, `PREPARE` 和`COMMIT`，也称为三阶段协议。

  - `PRE-PREPARE`阶段:  每次进入到一个新的round时，就会开始三阶段中的第一个阶段，即`PRE-PREPARE`阶段。在该阶段中，**Proposer**（区块提议者）节点生成一个提议区块，并广播给所有的validator节点。接着Proposer节点进入到PRE-PREPARED状态。其他validator 节点接收到有效的 `PRE-PREPARE` 消息后进入到`PRE-PREPARED` 状态。
  - `PREPARE`阶段: 在这一阶段，validator 节点广播`PREPARE`消息给其他validator 节点，并等待接收`+2/3` 的有效的 `PREPARE` 消息从而进入到`PREPARED`状态。
  - `COMMIT`阶段:  在这一阶段，validator 节点广播`COMMIT` 消息给其他validator 节点，并等待接收`+2/3` 的有效的 `COMMIT`  消息从而进入到 `COMMITTED` 状态。

  以上三阶段完成后，整个共识流程就成功完成了。

#### 3.1.4. 状态迁移:

  下图描述了PlatONE的共识流程的状态迁移过程。

  ![State transition diagram](imgs/state_flow.jpg)

  * `NEW ROUND` \-\> `PRE-PREPARED`: (对应于`2.1.3`节中的`PRE-PREPARE`阶段)
    - **Proposer**从txpool中收集交易。
    - **Proposer**生成一个提议区块并广播给其他validator节点，接着就进入到`PRE-PREPARED` 状态。
    - 每一个**validator**节点接收到满足如下条件的`PRE-PREPARE` 消息后，进入到`PRE-PREPARED`状态：
      - 提议区块来自于有效的proposer节点。
      - 区块头有效
      - 提议区块的sequence（高度）和round和**validator**节点的当前状态一致。
    - **Validator**节点广播`PREPARE` 消息给其他validator节点。
  * `PRE-PREPARED` -\> `PREPARED`: (对应于`2.1.3`节中的`PREPARE`阶段)
    - **Validator**接收到`+2/3` 的有效的 `PREPARE` 消息，从而进入到`PREPARED`状态。有效的消息需要满足如下条件：
      - sequence 和 round 相一致
      - 区块哈希一致
      - 消息来自于已知的validator节点
    - **Validator** 节点在进入到`PREPARED`状态后，广播`COMMIT`消息。
  * `PREPARED` -\> `COMMITTED`: (对应于`2.1.3`节中的`COMMIT`阶段)
    - **Validator**接收到`+2/3` 的有效的`COMMIT` 消息，从而进入到`COMMITTED` 状态。有效的消息需要满足如下条件：
      - sequence 和 round 相一致
      - 区块哈希一致
      - 消息来自于已知的validator节点
  * `COMMITTED` -\> `FINAL COMMITTED`:
    - **Validator**节点将`+2/3`的commitment签名（committed seal）添加到区块头的`extraData`字段中，并尝试将区块插入到区块链中。
    - 区块上链成功后，**Validator**节点进入到`FINAL COMMITTED` 状态。
  * `FINAL COMMITTED` -\> `NEW ROUND`:
    - 各**Validator**节点选取出一个新的**proposer**节点，并启动一个新的round定时器。

### 3.2. Round change 机制

以下三种条件都会触发`ROUND CHANGE`:

- Round change定时器超时触发
- 无效的`PREPREPARE`消息
- 区块上链失败

#### 3.2.1. round change 的流程

* 当一个validator节点检测到以上round change触发条件之一满足时，将会广播`ROUND CHANGE`消息，其中包含要变更到的目标round数值，同时等待接收来自其他validator节点的`ROUND CHANGE`消息。目标round的数值基于以下条件选取：
  - 如果validator节点已经从其他peer节点接收到了 `ROUND CHANGE` 消息，则从所有数量达到`F + 1` 的`ROUND CHANGE` 消息中包含的round数值中选取出最大的那个数值
  - 否则，将目标round的数值设置为：当前的round数值+1
* 任何时候，如果一个validator节点接收到了`F + 1` 条含有相同的目标round数值的 `ROUND CHANGE` 消息，就会将该round数值和其自己的进行比较。如果接收到的数值更大，validator节点就再次广播`ROUND CHANGE` 消息，而消息中的round数值和接收到的相同。
* 一旦validator节点接收到了`2F + 1` 条带有相同round数值的 `ROUND CHANGE` 消息，则结束round change循环，确定出新的**proposer**节点，之后进入到`NEW ROUND`状态。
* 触发validator节点退出round change循环的另外一个条件是其通过p2p同步机制同步到验证后的区块。

### 3.3. 区块锁定机制

* 锁定区块的触发条件

  节点`锁定`在区块`B`、`round number` `R` 的含义是指，当前节点**只能**对区块`B`的信息投`commit`票 。当一个节点收到了`+2/3`个对区块`B`的`PREPARE`投票后，进入`PREPARED`状态。此时，节点被锁定，等待接收其他节点的`commit`投票信息，锁定的round即当前round；

* 锁定区块的机制

  除了共识起始阶段，当收到更高区块的同步数据时，或当前高度成功产生区块并达成共识时，锁定被状态重置为非锁定状态，并开始新一轮对更高区块共识。如未能在锁定期间收到`+2/3`个指定round和区块的`commit`投票，则触发`ROUND CHANGE`。并且，在特定场景下，原有锁定解锁机制还会出现死锁的情况，我们在代码层面也优化了相关的解锁实现。具体可参考「2. 对Istanbul锁定解锁机制的优化」。

### 3.4. Consensus proof 目前的存储机制

 区块上链前，每个validator节点需要收集`2F + 1`个committed seal以构成一个consensus proof（共识证明）。一旦validator节点接收到足够的committed seal，就会将其存储于区块头的`extraData`字段中IstabulExtra结构中`CommittedSeal` 字段中，并重新计算`extraData`字段，然后将区块插入到区块链中。

Committed seal计算过程如下：

* Committed seal的计算:

  每个validator节点使用其私钥对区块哈希级联上commit消息代码`COMMIT_MSG_CODE`的结果进行签名，得到签名即为Committed seal：

  - `Committed seal`: `SignECDSA(Keccak256(CONCAT(Hash, COMMIT_MSG_CODE)), PrivateKey)`
  - `CONCAT(Hash, COMMIT_MSG_CODE)`: 将区块哈希和commit消息代码`COMMIT_MSG_CODE` 进行级联
  - `PrivateKey`: 进行签名的validator节点的私钥

* 上面提到的`extraData`是区块头的一个字段，其数据组成为：EXTRA_VANITY | ISTANBUL_EXTRA，其中|用以表示分隔EXTRA_VANITY和ISTANBUL_EXTRA的固定的索引（不是一个实际的分隔字符）。

* IstabulExtra结构的类型定义如下：

```go
  type IstanbulExtra struct {
  Validators    []common.Address 	//Validator addresses
  Seal          []byte			//Proposer seal 65 bytes
  CommittedSeal [][]byte			//Committed seal, 65 * len(Validators) bytes
  }
```
   其中，各字段的含义如下：
   + Validators：参与共识的各validator节点的列表
   + Seal：Proposer 节点对区块的签名，长度为65字节
   + CommittedSeal：用于存储validator节点收集到的committed seal列表

