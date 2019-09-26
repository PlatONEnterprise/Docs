​    The new user submits the registration information and becomes a platform user after the chain administrator has approved it. The user can query whether the user information is legal, update the user information, disable and activate the user, and delete the user through the user address and the user name.

​	Platform users can apply for different roles. After the chain administrator has passed the audit, platform users are granted the corresponding roles and have the corresponding permissions. Users can query and revoke roles through role management contracts, use the  address and  name to query user role information , whether there is a role, address of a role in the system, or revoke user role.

The permissions in PlatONE are assigned to users in the system. The user's role represents the user's permissions. Currently, users can assign the following roles (permissions).

| User role (permission) | Permissions                                             |
| :--------------- | :----------------------------------------------- |
| chainCreator     | The chain creator, generated when the chain is created, is the highest-privileged account in the system. |
| chainAdmin       | Chain administrator, set by chain creator, can set multiple chain administrators|
| nodeAdmin        | Node administrator for managing node information in the system             |
| contractAdmin    | Contract administrator, who can manage contract-related access control in the system   |
| contractDeployer | Chain Deployer, this role indicates that the user can deploy the contract on the chain|

The scope of permissions for each role is shown in the following table.

|                            | chainCreator | chainAdmin | nodeAdmin | contractAdmin | contractDeployer |
| -------------------------- | ---------------- | ---------------- | ---------- | ---------- | ---------- |
| Specify or cancel a chain administrator   | &radic;          |                  |            |            |            |
| Specify or cancel a node administrator       | &radic;          | &radic;          |            |            |            |
| Specify or cancel a contract administrator       | &radic;          | &radic;          |            |            |            |
| Specify or cancel a contract deployer       | &radic;          | &radic;          |            | &radic;    |            |
| New node application               | &radic;          | &radic;          | &radic;    |            |            |
| Manage all node             | &radic;          | &radic;          |            |            |            |
| Set up a firewall for your own deployed contract | &radic;          | &radic;          |            |            | &radic;    |
| Review deployed contracts           | &radic;          | &radic;          |            | &radic;    |            |
| Manage the nodes you joined         | &radic;          | &radic;          | &radic;    |            |            |
| Deploy contract                   | &radic;          | &radic;          |            |            | &radic;    |


## Node Management

In order to ensure the security of the blockchain, PlatONE manages the multi-faceted operations of nodes such as joining and exiting nodes. The node management function is based on the **nodeManager system contract**, which introduces two node types and two node states and node updates.

### Node type

#### Observer node

Observer node responsibilities:

- Responsible for syncing blocks but not participating in voting.

There will always be several stable observer nodes in the system. Used for other node connections to make stable sync blocks. When the new node starts, use the bootnodes parameter to specify the ** stable observer node** to connect. After the connection is successful, the block can be synchronized normally. The stable observer nodes are set in the suggestObserverNodes of genesis.json. After the chain is started, the set stable observer nodes need to be added to the node management system contract.

#### Consensus node

Consensus node responsibilities:

- Participate in voting, and sync blocks

At the beginning of PlatONE, the system contract has not been deployed, and the consensus node in the network will be assumed by the first node specified in the validatorNodes in genesis.json. This node makes a consensus out of the block. Until PlatONE has deployed the system contract and successfully assigned a new consensus node in the node management contract, then multiple consensus nodes specified in the contract vote and vote out each other.

### Node status

Node status can be set and changed using the corresponding API of the node management system contract.

#### Delete status

The node in the deleted state cannot connect to other nodes, and cannot be connected by other nodes. The node in the deleted state will stay in the deleted state forever, and recovery is not allowed.

#### Normal state

A node in a normal state can actively connect to other nodes, or can be connected by other nodes in a normal state.

### Node startup

When the node starts, in addition to specifying the PlatONE general parameters, you need to specify `--bootnodes`. The specified range of bootnodes is recommended choice  stable observer nodes. All stable observer nodes can be viewed in suggestObserverNodes in genesis.json.

### Node connection

After node A starts, it will actively connect to node B specified by `--bootnodes`, and in node B, it will check if node A is present in the node management contract, and the node status is normal. Only when the check passes, Node B will accept the connection request of Node A. After the node is successfully connected, it can perform operations such as synchronizing blocks.

### Node update

The node management contract provides the corresponding node update operation. You can change the node status, type, and other node information of the added nodes in the contract as needed.


## Contract Firewall

​	The contract firewall mainly implements the management function of the whitelist and blacklist of the firewall, so that the contract deployer can set the firewall rules of the contract to control the calling permission of the contract method, and allow specific users to call the specified method of the contract.It not only improves the user experience, but also effectively improves the security performance.


