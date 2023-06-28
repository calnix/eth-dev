# What is a smart contract

Coined by computer scientist Nick Szabo in 1994.

SC is a piece of code running on the blockchain. The blockchain is a state machine (EVM) - transactions are required to change state. The computing costs for affecting such a change are gatscosts, paid in ETH.

State change is achieved through mining + transactions. Mining is for verification and committing changes.

The bytecode of the SC is stored on the blockchain, regardless if it was written in Solidity or otherwise.

Every Ethereum node in the network will execute submitted transactions for verification and thereafter consensus.&#x20;

### Structure of Solidity

* Class-like structure
* Functions
* Control structures: If/else
* Loops: For/while
* Data Types:&#x20;
  * uint, int, Boolean, Address
  * Mapping, Array, Struct
  * No floats!
* Inheritable
* Modifiers&#x20;
* Imports

### Why is contract code immutable?

The reason contract code is immutable, but the variable is not, is that there are opcodes in the EVM that allow changing of the contract's "storage", but none that modify the code. So, there is a valid transaction you can make that will cause every node in the network to update the variable to the new value in their copy of the world state. But there's no valid transaction you can make that will cause other nodes to update the contract code in their world state.

There are more details about this stuff in [the formal specification](http://yellowpaper.io/). See section 4 for more information. Specifically, check out the `storageRoot` and `codeHash` properties of account state in section 4.1, and the `stateRoot` property of blocks, in section 4.4.

#### Addon

Each contract has a private storage, meaning only the contract has access to write to them. This storage is protected by [Merkle Patricia Trees](https://github.com/ethereum/wiki/wiki/Patricia-Tree).

The idea behind the algorithm is that the minimum modification of flipping a single bit of storage will generate a different storageRoot and everybody will notice immediately.

Each contract storageRoot is stored in something knows as the "World State", this also includes addresses' balances, contract's code, etc. This "World State" is protected again with a Merkle Patricia Tree. The result after applying all transactions in a block is recorded in the block as stateRoot.
