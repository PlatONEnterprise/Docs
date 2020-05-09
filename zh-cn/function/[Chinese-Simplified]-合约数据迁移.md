## 选型

在数据合约升级的场景，某些情况需要处理历史数据在新旧合约之间的迁移。迁移的方法有如下三种，各有特点。

### 硬编码迁移法

硬编码迁移法指的是，新版本的数据合约中保存一个指向旧版本数据合约的合约地址，新版本数据合约保存的是增量的数据内容。

这样相当于新版本合约保留了一份旧版本数据的指针，当新版本需要使用旧数据的时候，直接调用旧数据合约地址对应数据接口即可。这样，新旧版本数据合约可以并存，即使是在异常情况下，数据被误写到了旧版本合约上，它依然可以被新版本所访问到。

这个方法的优点是：新旧合约可以同时并存，不增加区块链存储压力，简单灵活，较强的升级容错能力。缺点：持续不断的版本升级会导致形成较长的链式逻辑关系，维护成本较高。

### 硬拷贝迁移法

硬拷贝迁移法指的是，新版本和旧版本之间切断逻辑关系，利用外部迁移工具，将旧版本数据逐步拷贝到链下，再从链下重新存储到新版本合约的过程。

这个方法的优点是：无历史包袱。缺点是：大幅度增加区块链存储压力；数据迁移工具需要适配不同的数据合约，开发成本较高；迁移过程需要停止服务，否则容易出现脏数据；数据量大时，耗时长，操作复杂，容易出错，基本无法实操。

### 默克尔树迁移法

默克尔数迁移法要点如下：

1. 利用智能合约语言的面向对象的继承特性，使得新版本合约存储结构完全兼容旧版本合约存储结构。
2. 利用智能合约在区块链上的storage树原理，使得新版本合约的storeage树直接从旧版本合约上衍生。无需显式的迁移过程。
3. 利用区块链交易的原子性，使得新版本合约的部署、数据迁移、升级，原子完成。

这个方法拥有前面两个方法的所有优点，且简单高效，安全，实操性强。缺点：需要区块链底层功能特性的支持。

### 比较小结

比较了上述方案效率及操作可行性，我们优先考虑采用默克尔树迁移法，优势在于：

* 迁移效率最高，用户迁移成本最小；
* 避免导入导出导致的迁移错误；
* 灵活性较高，无需引入额外的硬编码；



## 整体方案
### 整体描述

每个合约账户对应一个`stateObject` ，`stateObject`的成员变量 `account`包含了存储树根哈希 `Root`，根据`Root`可以索引到账户所有的存储数据。

```go
type Account struct {
   Nonce    uint64
   FwActive  uint64
   Balance  *big.Int
   Root     common.Hash // merkle root of the storage trie
   CodeHash []byte
   AbiHash  []byte
   Creator   common.Address
   FwDataHash []byte
}
```

因此，考虑将新的数据合约C<sub>New</sub>中对应的`Root`字段，指向升级前的数据合约C<sub>Old</sub>中的Root值，从而在底层实现数据迁移。

由于在PlatONE中，存储树storage trie中实际存储的键为`keytrie`的哈希，而keytrie由`address`、实际的`key`组成。新的数据合约对应新的合约地址`adresss`，因此不可将`Root`直接指向旧合约C<sub>Old</sub>的Root，需要根据旧合约C<sub>old</sub>的存储树重建一颗新的存储树，即以下过程：

* 遍历旧合约C<sub>Old</sub>的存储树；
* 替换每一个`keytire`中的地址为新合约的地址；
* 用上述遍历对象构造一颗新树；
* 将C<sub>New</sub>. stateObject.Account.Root 指向这个新生成的树根。



存储树storage trie的迭代过程，参考core/state/dump.go 的 RawDump()函数：

```go
func (self *StateDB) RawDump() Dump {
   dump := Dump{
      Root:     fmt.Sprintf("%x"，self.trie.Hash())，
      Accounts: make(map[string]DumpAccount)，
   }

   it := trie.NewIterator(self.trie.NodeIterator(nil))
   for it.Next() {
      addr := self.trie.GetKey(it.Key)
      var data Account
      if err := rlp.DecodeBytes(it.Value，&data); err != nil {
         panic(err)
      }

      obj := newObject(nil，common.BytesToAddress(addr)，data)
      account := DumpAccount{
         Balance:  data.Balance.String()，
         Nonce:    data.Nonce，
         Root:     common.Bytes2Hex(data.Root[:])，
         CodeHash: common.Bytes2Hex(data.CodeHash)，
         Code:     common.Bytes2Hex(obj.Code(self.db))，
         Storage:  make(map[string]string)，
      }
      storageIt := trie.NewIterator(obj.getTrie(self.db).NodeIterator(nil))
      for storageIt.Next() {
         account.Storage[common.Bytes2Hex(self.trie.GetKey(storageIt.Key))] = common.Bytes2Hex(storageIt.Value)
      }
      dump.Accounts[common.Bytes2Hex(addr)] = account
   }
   return dump
}
```


### 实施方案

#### 迁移发起

1. 参考合约防火墙方案，

   * 在`ctool/contractcmd.go `中添加`migInvoke(fromAddr String，toAddr String)`方法。

   * 参数指定需要迁移数据的老合约，以及需要迁入数据的新合约。

   * 同时添加`txType`类型：`MigrateTxType`，用以指示这个交易的类型为数据迁移操作。

   * 此外需要检查权限：两个合约的地址的管理者是否相同。

2. 调用`MigrateInvokeContract()`构造调用交易，并调用`Send()`发送交易。

#### 迁移交易的处理

1. 在`state_transation.go`中`TransitionDb()`函数中，添加对类型为`MigrateTxType`的交易的拦截处理：

   ```
   if msg.TxType() == types.MigrateTxType {
   	ret，st.gas，vmerr = migProcess(evm.StateDB，st.to()，msg.From()，msg.Data())
   } 
   ```

2. 在`migProcess()`中处理以下逻辑：

   * 新建`trie`，需要在state_object中添加`newTrie()` 方法；

   * 迭代老合约`old_trie`，迭代处理`old_trie`的 `old_node`:

   ​	- 新建`new_node`，每个`new_node`的`key`是替换`old_node` 中的prefix为"新合约的地址"后的值；

   ​	- append`new_node`到`newTrie`上；

   * 将新合约的stateObject.Account.Root 指向`newTrie`树根。

     

   备注：迁移数据在新合约上的兼容性，需要合约逻辑保证。