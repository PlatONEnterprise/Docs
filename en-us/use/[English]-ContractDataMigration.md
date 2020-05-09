## Operation Command

```
./ctool migInvoke --addr ${destination_contract_addr} --func 'migrateFrom' --param ${source_contract_addr}  --config ${ctool.json}
```

Parameter Description:

`${destination_contract_addr}` is the contract address where data is moved into;

`${source_contract_addr}` is the data source contract from where to be migrated from.

## Examples

Let's describe the operation process by migrating a CNS contract with data as an example:

1. Deploy the CNS contract A:

```
./ctool deploy --code ../conf/contracts/cnsManager.wasm --abi ../conf/contracts/cnsManager.cpp.abi.json --config ../conf/ctool.json
```

2. Deploy the `userRegister` contract to write data in CNS Contract A. (In the user scenario, this step is optional, as long as the data of the contract to be migrated exists.)

```
./ctool deploy --code ../conf/contracts/userRegister.wasm --abi ../conf/contracts/userRegister.cpp.abi.json --config ../conf/ctool.json
```

3. Deploy the CNS contract B:

```
./ctool deploy --code ../conf/contracts/cnsManager.wasm --abi ../conf/contracts/cnsManager.cpp.abi.json --config ../conf/ctool.json
```

4. Perform migration (copy, no impact on the original contract data):

```
./ctool migInvoke --addr 0x30c9f12cae592610df1a387cfec39db0e64989e4 --func 'migrateFrom("0xe1dcc86f53fcbad47e25e391111b03afc14f1db3")' --config ../conf/ctool.json
```

5. Review the data on CNS Contract B to confirm that the data has been migrated:

```
./ctool invoke --addr 0x30c9f12cae592610df1a387cfec39db0e64989e4 --abi ../conf/contracts/cnsManager.cpp.abi.json --config ../conf/ctool.json --func getRegisteredContracts --param 0 --param 10
```