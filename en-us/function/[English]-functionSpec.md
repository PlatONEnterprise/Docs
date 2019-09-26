## Brief introduction to Wasm Contracts

The smart contract system used in the `PlatONE` platform is completely different from traditional smart contracts. The transaction that triggers the Wasm contract is packaged by the consensus node, and the entire network node repeats the verification. The status of the Wasm contract is kept in the public ledger. The Wasm contracts use `C++` syntax. A large class library - the Wasm contract built-in library is provided in order to simplify the process of integrating with the PlatONE platform and writing high-performance, powerful smart contracts.

## The PlatONE virtual machine

The PlatONE virtual machine `BCWasm` is the operating environment for the Wasm contracts. Executed in a completely isolated sandbox, the contract logic is not able to access the network, file system, or any other external resource. Even access between Wasm contracts is limited.

This document is mainly an introduction to Wasm contract development guide under Centos. Development guides for other environments will be released very soon, so stay tuned!

