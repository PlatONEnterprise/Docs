In the current mainstream blockchains, users access smart contracts by address, such as Ethereum. The address of the smart contract is a hexadecimal string that users need to remember in order to access the smart contract. When users want to upgrade the contract, the redeployment of the new contract generates a new address, and all modules that depend on the contract need to be updated accordingly. Obviously, the existing way to access the contract is not friendly to users, so we have implemented the CNS in PlatONE. Users can access the smart contract through the contract name and version number.

CNS maintains the mapping relationship from name, version to contract address, and provides contract management functions in the system, including contract registration and cancellation, contract registration information and address query and other functions.

PlatONE implements the CNS with the system contract. After the contract is deployed, the user can register the contract into the system contract. The subsequent calls can be made through the contract name and version, instead of using the contract address. If the transaction calls the contract through the contract name and version, the underlying PlatONE will automatically query the contract address corresponding to the contract name version in the system contract, and then call the contract at that address.