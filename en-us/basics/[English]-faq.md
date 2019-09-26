* **1. What is the application of PlatONE?**

    PlatONE is a financial-grade blockchain platform. It has involved in areas like payment and settlement, AI, IoT, healthcare, credit system,advertising and key management.

* **2. Which consensus algorithms do you use and what is the time needed to achieve consensus?**

    PlatONE supports pluggable different consensus protocol which includes Concurrent BFT and Optimized BFT. VRF and probability distribution are used to randomly select consensus nodes, which can balance decentralization and scalability.

    - Concurrent BFT: block generation and block verification are performed in parallel, which greatly improves the rate of block generation while ensuring BFT 1/3 fault tolerance. In the test network, it takes 1 second to reach consensus and generate block.
    - Optimized BFT: Introducing unlocking mechanism to solve the consensus deadlock issue and the chain can support more than 100 consensus nodes. In the test network, it takes 1 second to reach consensus and generate block.

* **3. What is the strengths and advantages of PlatONE?**

    We introduce the latest technology to provide  stable and high performance platform. It has the following characteristics:

    - Privacy and confidentiality mechanisms: Introducing SMC(Secure Multi-Party Computation), homomorphic encryption, SM series encryption algorithm,distributed key management and other cryptographic algorithms to ensure the security of shared data under the premise of privacy protection. 
    - Consensus: Using a highly optimized BFT consensus, supporting more than 100 nodes, Preventing Byzantine failures and ensuring security of blockchain, greatly improving the degree of decentralization while retaining the features of instant finality. Adding unlock mechanism to solve deadlock problem of several well-known projects.
    - Operation Management: System configuration parameters are managed by contracts, supporting technology upgrades and management.
    - Contract VM: Supporting both EVM and Wasm (WebAssembly), users can alternatively choose when deploying PlatONE, friendly to developers.

* **4. How to start the TestNet?**

    Now we provide two tutorials for installation instruction:

    - manual start: detail see section 1.4.1 .

    - quick start: support quickly deploy and start the chain, details see section 1.4.2 .

* **5. How many parties does MPC support?**

    PlatONE will first support secure two-party computation and then support  multi-party computation. Theoretically, there is no limit to the number of parties supported by secure multi-party computation.

* **6. What is the requirement for connecting to the TestNet?**

    There is no special requirements for computing power while running the PlatONE nodes, but make sure that you have as much disk as possible to accommodate the growing needs of blockchain data in the future.

    Minimum configuration for node server:

    - CPU: 2 cores
    - memory: 2G
    - disk: 100G

    PlatONE currently only released the Go version, the software requirements are as follows:

    - System：Linux
    - Golang：Go 1.11+
    - Cmake：3.10+
    - gcc：7.3+

* **7. What is the difference between smart contract on PlatONE and Ethereum?**

    Unlike Ethereum smart contracts mainly written in solidity, PlatONE’s smart contracts are divided into three categories.

    Wasm Contracts support high-level language development and compiles to executable Wasm. Transactions that trigger wasm contracts are packaged by the consensus nodes, and the nodes in the entire network repeat the verification. The statuses of the wasm contracts are saved to the public ledger.

    Verifiable Contracts are developed and published in the same way as wasm contracts; they also compile to executable Wasm. The state transitions of verifiable contracts are asynchronously executed by computation nodes off the chain. After a computation is completed, the new state and the state transition proof are submitted to the chain, and the entire network of nodes can quickly verify the correctness and update the new state to the public ledger. Verifiable contracts support complex, heavy computing logic without affecting the performance of the entire chain.

    Privacy Contracts also support high-level language development and compile to executable Intermediate Language (LLVM IR). The input data to privacy contracts is stored locally on data nodes. The data nodes perform private computation off the chain using secure multiparty computation, and submit the computation result to the chain.

* **8. Which programming languages do you use?**

    We use Golang to develop the underlying of the chain. Besides we use C++, Python, Go, Rust, Solidity, Java and other language to develop the smart contracts. In addition, Java is used for SDKs and applications.

* **9.What are the advantages of the deployment and operations toolset?**

    At present, the system provides a rich set of enterprise-level deployment tools, which greatly improves the user's ease of use and reduces learning costs. Supports multi-node startup with one button, and provides rich operation and maintenance scripts, which greatly reduces the difficulty of operation and maintenance.

* **10. What is the function of the different node types?**

    There are two kinds of nodes, namely observer nodes and consensus nodes. The observer node is only responsible for the synchronization blocks and does not participate in the block generation. In the system, there will always be several stable observer nodes. It is used to stabilize the synchronization block and acts as bootnodes to be connected by other nodes. While the consensus node participate in the block generation and block synchronization.

* **11. What is the authority management?**

    PlatONE modularizes the authority management based on the different entity objects in the system. The user roles management module, node management module and contract firewall module is designed by the different behaviors of user accounts, nodes and smart contracts in the system.

    - Role management: PlatONE sets different user roles according to different permissions, and manages the user's role through system contract.  Different roles' permissions are various in the system.
    - Node management: PlatONE manages nodes through node management contracts in which node can access the network, participate in the consensus and maintain the node information. According to the previous roles settings, only the ChainCreator, chainAdmin and nodeAdmin can set the node data in the system contract. When you want to add nodes, update node status or delete nodes, you need these three types of accounts to invoke node management contract.
    - Contract firewall: The invoking permissions of contracts are controlled by the contract firewall. Only the creator of the contract can set the contract firewall which can control the access of contract interfaces.

* **12. How to introduce contracts into CNS management?**

    When the contract is deployed, the CNS contract interface is called, and the contract name, version, and address information will be written into the CNS table.


* **13. What can I do for PlatONE?**

    PlatONE is still in the development and needs further improvement. Therefore, we truly hope that all parties can participate in this project. You can participate in the construction of PlatONE from the following aspects:

    - Participate in PlatONE's functional iterations, bug fixes, solutions, etc.
    - Participate in PlatONE technical documentation, translation, soft text promotion, UI poster design, etc.
    - Participate in the community building of PlatONE, organizing events and so on.

* **14. What can I do to become a member of the GitHub project team at PlatONE?**

    PlatONE welcomes community to submit proposals and codes via Issues and Pull Request via GitHub. The PlatONE core development team will evaluate the contributions and invite community elites to join the project team.