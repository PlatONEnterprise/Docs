## Selection

In the scenario of data contract upgrades, in some cases it is necessary to handle the migration of historical data between new and old contracts. There are three ways to migrate, each with its own characteristics.

### Hard Code Migration Method

The hard-coded migration method means that the new version of the data contract holds a contract address pointing to the old version of the data contract, and the new version of the data contract holds the incremental data content.

This is equivalent to the new version of the contract retains a pointer to the old version of the data, when the new version needs to use the old data, directly call the old data contract address corresponding to the data interface. In this way, the old and new versions of the data contract can coexist, even if the data is mistakenly written to the old version of the contract under abnormal circumstances, it can still be accessed by the new version.

The advantage of this method is that the old and new contracts can coexist at the same time, without increasing the blockchain storage pressure, simple and flexible, and strong upgrade fault tolerance. Disadvantages: Continuous version upgrades result in longer chained logic relationships and higher maintenance costs.

### Hard copy migration method

The hard copy migration method refers to the process of cutting off the logical relationship between the new version and the old version, using the external migration tool to gradually copy the old version data to the chain, and then re-storing from the chain to the new version of the contract.

The advantage of this method is: no historical burden. The disadvantages are: greatly increase the storage pressure of the blockchain; the data migration tool needs to adapt to different data contracts, and the development cost is high; the migration process needs to stop the service, otherwise the dirty data is prone to occur; when the data volume is large, it takes a long time and the operation is complicated. It is easy to make mistakes and basically cannot be practiced.

### Merkel Tree Migration Method

The main points of the Merkel number migration method are as follows:

1. Utilizing the object-oriented inheritance feature of the smart contract language, the new version of the contract storage structure is fully compatible with the old version of the contract storage structure.
2. Using the smart tree's storage tree principle on the blockchain, the storeage tree of the new version of the contract is derived directly from the old version of the contract. No explicit migration process is required.
3. Using the atomicity of blockchain trading, the deployment, data migration, upgrade, and atomic completion of the new version of the contract.

This method has all the advantages of the first two methods, and is simple, efficient, safe, and practical. Disadvantages: Support for the underlying functional features of the blockchain.

### Comparison Summary

Comparing the efficiency and operational feasibility of the above scheme, we give priority to adopting the Merkel tree migration method described in Section 1.3. The advantages are:

* The highest migration efficiency and the lowest user migration cost;
* Avoid migration errors caused by import and export;
* High flexibility without the need to introduce additional hard coding;

## Overall Plan

### Overall description

Each contract account corresponds to a `stateObject`, and the member variable `account` of `stateObject` contains the storage tree root hash `Root`, which can be indexed to all the stored data of the account according to `Root`.

```go
Type Account struct {
   Nonce uint64
   FwActive uint64
   Balance *big.Int
   Root common.Hash // merkle root of the storage trie
   CodeHash []byte
   AbiHash []byte
   Creator common.Address
   FwDataHash []byte
}
```

Therefore, consider the corresponding 'Root` field in the new data contract C<sub>New</sub>, pointing to the Root value in the data contract C<sub>Old</sub> before the upgrade, thus implementing the data at the bottom layer. migrate.

Since in PlatONE, the key stored in the storage tree storage trie is a hash of `keytrie`, and the keytrie consists of `address` and the actual `key`. The new data contract corresponds to the new contract address `adresss`, so you can't point the `Root` directly to the root of the old contract C<sub>Old</sub>, which needs to be stored according to the old contract C<sub>old</sub> The tree reconstructs a new storage tree, which is the following process:

* Traverse the storage tree of the old contract C<sub>Old</sub>;
* Replace the address in each `keytire` with the address of the new contract;
* Construct a new tree with the above traversal object;
* Point C<sub>New</sub>. stateObject.Account.Root to this newly generated tree root;



For the iterative process of the storage tree storage trie, refer to the RawDump() function of core/state/dump.go:

```go
Func (self *StateDB) RawDump() Dump {
   Dump := Dump{
      Root: fmt.Sprintf("%x", self.trie.Hash()),
      Accounts: make(map[string]DumpAccount),
   }

   It := trie.NewIterator(self.trie.NodeIterator(nil))
   For it.Next() {
      Addr := self.trie.GetKey(it.Key)
      Var data Account
      If err := rlp.DecodeBytes(it.Value, &data); err != nil {
         Panic(err)
      }

      Obj := newObject(nil, common.BytesToAddress(addr), data)
      Account := DumpAccount{
         Balance: data.Balance.String(),
         Nonce: data.Nonce,
         Root: common.Bytes2Hex(data.Root[:]),
         CodeHash: common.Bytes2Hex(data.CodeHash),
         Code: common.Bytes2Hex(obj.Code(self.db)),
         Storage: make(map[string]string),
      }
      storageIt := trie.NewIterator(obj.getTrie(self.db).NodeIterator(nil))
      For storageIt.Next() {
         account.Storage[common.Bytes2Hex(self.trie.GetKey(storageIt.Key))] = common.Bytes2Hex(storageIt.Value)
      }
      dump.Accounts[common.Bytes2Hex(addr)] = account
   }
   Return dump
}
```



### Implementation plan

#### Migration initiation

1. Refer to the contract firewall solution,

   * Add the `migInvoke(fromAddr String, toAddr String)` method to `ctool/contractcmd.go `.

   * The parameter specifies the old contract that needs to migrate the data, as well as the new contract that needs to move in the data.

   * Also add the `txType` type: `MigrateTxType` to indicate that the type of this transaction is a data migration operation.

   * In addition, you need to check the permissions: whether the managers of the addresses of the two contracts are the same.

2. Call the `MigrateInvokeContract()` construct to call the transaction and call `Send()` to send the transaction.

#### Processing of migration transactions

1. In the `TransitionDb()` function in `state_transation.go`, add the interception of the transaction of type `MigrateTxType`:

   ```
   If msg.TxType() == types.MigrateTxType {
   Ret, st.gas, vmerr = migProcess(evm.StateDB, st.to(), msg.From(), msg.Data())
   }
   ```

2. Process the following logic in `migProcess()`:

   * New `trie`, need to add `newTrie()` method to state_object

   * Iterate over the old contract `old_trie` and iterate over the `old_node` of `old_trie`:

   - Create new `new_node`, the `key` of each `new_node` is the value after replacing the prefix in `old_node` with the address of "new contract";

   - append`new_node` to `newTrie`;

   * Point the stateObject.Account.Root of the new contract to the `newTrie` tree root;

     

   Note: The compatibility of migration data on new contracts requires contractual logic guarantees;


