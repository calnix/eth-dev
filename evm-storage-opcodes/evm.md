# EVM

## Accounts

* An account is an object in the world state.
* An account is a mapping between an Address and Account state
* Account state contains
  * nonce, balance, storage hash, code hash

![](<../.gitbook/assets/image (93).png>) <img src="../.gitbook/assets/image (162).png" alt="" data-size="original">

### Two types of accounts

![](<../.gitbook/assets/image (74).png>)![](<../.gitbook/assets/image (160).png>)

* EOA controlled by private key and cannot contain EVM code
* Contract Account (CA) contains EVM code
* You can determine if an address is a Solidity smart contract by checking the size of the code stored at the address.
* Assembly `extcodesize` is used in Solidity functions to determine the size of the code at a particular address. If the code size at the address is greater than 0 then the address is a smart contract.
* Smart contracts prevent other SC from calling their functions by implementing an account code size check.
  * code size check determines if the address interacting with the contract contain code, and if it does the function is not executed
  * [https://cryptomarketpool.com/bypass-solidity-contract-size-check/](https://cryptomarketpool.com/bypass-solidity-contract-size-check/)

## Transaction

* A transaction is a single cryptographically-signed instruction.
* It can either be a
  * contraction creation
  * message call

![](<../.gitbook/assets/image (150).png>)![](<../.gitbook/assets/image (186).png>)![](<../.gitbook/assets/image (183).png>)

If contract creation:

* **To** field in a transaction is 0 (not specified)
* **Data** is the init code fragment, including the contract binary.
  * init function is NOT stored on the blockchain as it is just used for setup (constructor)

If message call:

* **To** field has to be some 160 bits (20 bytes) address
* For [EOA](https://ethdocs.org/en/latest/glossary.html#eoa), the address is derived as the last 20 bytes of the public key controlling the account, e.g., `cd2a3d9f938e13cd947ec05abc7fe734df8dd826`.
*
  * This is a [hexadecimal](https://ethdocs.org/en/latest/glossary.html#hexadecimal) format (base 16 notation), which is often indicated explicitly by appending `0x` to the address.
  * Since each byte of the address is represented by 2 hex characters, a prefixed address is 42 characters long.

### Message

* Message comprises of **Data** (as a set of bytes) and **Value** (specified as Ether)
* A message can be triggered by a transaction or by EVM code

<figure><img src="../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (193).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

## Atomicity of Transaction

* A transaction is an atomic operation. Cannot divide or interrupt
* It is either **completed in entirety** or **nothing is done** → no halfway, middling state.
* Transactions cannot be overlapped, they must be executed sequentially.
* Transaction order is not guaranteed.
  * **Order of transactions in a block:** can be determined by Miners
  * **Order between blocks** is determined by a consensus algo: like PoW

<figure><img src="../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (180).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

## Ethereum Virtual Machine (EVM)

* EVM code is executed on the EVM
* The Ethereum Virtual Machine is the runtime environment for smart contracts in Ethereum
* It is stack-based and does not have registers.

<figure><img src="../.gitbook/assets/image (156).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

It uses [big endian](https://en.wikipedia.org/wiki/Endianness)  byte ordering for instructions, memory, and input data. [Words](https://en.wikipedia.org/wiki/Word\_\(computer\_architecture\))  in the EVM are 256 bits (32 bytes) wide.

💡 The word size is the maximum size that the _majority_ of operations work with.

On stacks, you **PUSH** data onto the top of it, **POP** data off, and apply instructions like ADD or MULT to the first few values that lay on top of it.

![](<../.gitbook/assets/image (72).png>)![](<../.gitbook/assets/image (84).png>)

### **Memory & Storage**

* The **Stack** - with maximum **1024 elements**. Each element is **256 bits** (1 word).
* **Memory** (volatile memory) - byte addressed linear memory.
* **Storage** (persistent memory) - key-value store of **256 bit to 256 bit pairs**.

![](<../.gitbook/assets/image (90).png>)![](<../.gitbook/assets/image (82).png>)

<figure><img src="../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

* All operations are performed on the stack
* **Memory** is like a **big array** (that’s the “linear” part).
  * It can be addressed (indexed into) at the **byte level** (return 8 bit values) or at the **word level** (return 256 bit values).
  * Memory is unlimited but constrained by gas fee requirements.
* Account storage is more like a **map**, pairing **256 bit keys** with **256 bit values** (words to words). Each location in storage is **0 initialized**.
* Both the stack and memory are **volatile** (deleted after contract execution), but storage is **persistent** and stored in Ethereum’s world state (sticks around after execution).
* Initially, all storage and memory are set to zero in the EVM.

![](<../.gitbook/assets/image (76).png>)![](<../.gitbook/assets/image (187).png>)

![](<../.gitbook/assets/image (149).png>)

* The program code is stored in **virtual read-only memory** (**virtual ROM**) that is accessible using the CODECOPY instruction.
  * The CODECOPY instruction copies the program code into the main memory.
* EVM code is the bytecode that the EVM can natively execute.

## Simple Example

There exist several implementations of the EVM specification, including the one embedded in Geth. We’ll use Geth as it is one of the most popular Ethereum clients and handily supports an EVM command line utility for easy invocation.

`$ git clone <https://github.com/ethereum/go-ethereum.git> $ cd go-ethereum $ make all`

This, in addition to building the rest of Geth from source, will leave you with an evm executable in the build folder. Feel free to copy this executable to wherever you’d like to work.

Opcode reference:[https://www.evm.codes/](https://www.evm.codes/),[https://ethervm.io/](https://ethervm.io/)

#### Hello, EVM!

Now that we have an EVM implementation setup, let’s write some code! First, create an empty easm file to edit and open it up in your favourite editor.

`$ touch hello.easm`

Let’s write some **EVM assembly**, which is a text-based human readable format that will be _assembled_ into EVM bytecode for us by geth’s `evm` binary.

```
PUSH 0x2
PUSH 0x2
MUL
STOP
```

* run `./evm compile hello.easm`.
* Should get the rather intractable looking string `600260020200` out.
* This is your evm assembly assembled into evm bytecode (in hexadecimal format)!

Now go ahead and try `./evm --debug run hello.easm`

```
#### TRACE ####
PUSH1           pc=00000000 gas=10000000000 cost=3

PUSH1           pc=00000002 gas=9999999997 cost=3
Stack:
00000000  0x2

MUL             pc=00000004 gas=9999999994 cost=5
Stack:
00000000  0x2
00000001  0x2

STOP            pc=00000005 gas=9999999989 cost=0
Stack:
00000000  0x4

#### LOGS ####
```

* The first value (`0x`) is our return value. We have not explicitly called the RETURN opcode, so this is empty.
* We can also see the execution trace of our bytecode, along with the program counter value (pc), current remaining gas, and gas cost of each instruction.
* Additionally we can see a full view of the stack and every step of execution.
* Notice that the stack is shown top to bottom (index 0 is the top most element).

From this trace we can see our bytecode first pushes the value “2” on top of the stack, and then pushes another “2”. It then calls “mult” which multiplies the first two values on the stack together and pushes the result (4). Finally, it calls stop, halting execution.

### Returning a Value

Looking at the [return opcode definition](https://ethervm.io/#F3), you can see that it returns values stored in memory (the expression form is `return memory[offset:offset+length]`). So, let’s write a value to memory!

As previously mentioned, memory is sequentially addressable. We’ll use the `MSTORE8` instruction, which sets memory at the index value on the top of the stack (we’ll call this _s_) to the second value on the stack (_s+1_).

```
PUSH 0x2
PUSH 0x0
MSTORE8
```

Basically we’re telling the EVM to store the 8 bit value “2” at location “0”.

Running this through the evm, we can now see a layout of memory too, with our 2 value sitting in the first byte. Nice!

```
Memory:
00000000  02 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
```

Next, let’s go ahead and try that return.

```
PUSH 0x1
PUSH 0x0
RETURN
```

This is telling the evm “go ahead and return memory values 0 to 0+1”. Running that, we get a return value of `0x2`. Perfect!

<figure><img src="../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

## Message Call

<figure><img src="../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

![](<../.gitbook/assets/image (86).png>)![](<../.gitbook/assets/image (169).png>)

* Every time a Solidity contract calls a function of another contract, it does so by producing a message call.
* Every call has a sender, a recipient, a payload, a value, and an amount of gas. The depth of the message call is limited to less than 1024 levels.
* `address.call.gas(gas).value(value)(data)`
  * `gas` is the amount of gas to be forwarded
  * `address` is the address to be called
  * `value` is the amount of Ether to be transferred in **Wei**
  * `data` is the payload to be sent
  * gas and value are optional parameters
    * → be careful because almost all the remaining `gas` of the sender will be sent by default in a low-level call.
    * Given that every call can end in an out-of-gas (OOG) exception, to avoid security issues at least 1/64th of the sender’s remaining gas will be saved. This allows senders to handle inner calls’ out-of-gas errors so that they are able to finish its execution without themselves running out of gas, and thus bubbling the exception up.

#### Memory

Memory is a volatile read-write byte-addressable space. It is mainly used to store data during execution, mostly for passing arguments to internal functions. Given this is a volatile area, every message call starts with a cleared memory. All locations are initially defined as zero. As calldata, memory can be addressed at the byte level, but can only read 32-byte words at a time.

<figure><img src="../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

Memory is said to “expand” when we write to a word in it that was not previously used. Additionally to the cost of the write itself, there is a cost to this expansion, which increases linearly for the first 724 bytes and quadratically after that.

The EVM provides three opcodes to interact with the memory area:

* `MLOAD` loads a word from memory into the stack.
* `MSTORE` \*\*saves a word to memory.
* `MSTORE8` saves a byte to memory.

Solidity also provides an inline assembly version of these opcodes.

There is another key thing we need to know about memory. Solidity always stores a free memory pointer at position `0x40`, i.e. a reference to the first unused word in memory. That’s why we load this word to operate with inline assembly. Since the initial 64 bytes of memory is reserved for the EVM, this is how we can ensure that we are not overwriting memory that is used internally by Solidity. For instance, in the `delegatecall` example presented above, we were loading this pointer to store the given `calldata` to forward it. This is because the inline-assembly opcode `delegatecall` needs to fetch its payload from memory.

Additionally, if you pay attention to the bytecode output by the Solidity compiler, you will notice that all of them start with 0x6060604052…, which means:

```
PUSH1   :  EVM opcode is 0x60
0x60    :  The free memory pointer
PUSH1   :  EVM opcode is 0x60
0x40    :  Memory position for the free memory pointer
MSTORE  :  EVM opcode is 0x52
```

<mark style="color:red;">**You must be very careful when operating with memory at assembly level. Otherwise, you could overwrite a reserved space.**</mark>

Big Endian for memory (page 69):[https://takenobu-hs.github.io/downloads/ethereum\_evm\_illustrated.pdf](https://takenobu-hs.github.io/downloads/ethereum\_evm\_illustrated.pdf)

### Storage

Storage is a persistent read-write word-addressable space. This is where each contract stores its persistent information. Unlike memory, storage is a persistent area and can only be addressed by words. It is a key-value mapping of 2²⁵⁶ slots of 32 bytes each. A contract can neither read nor write to any storage apart from its own. All locations are initially defined as zero.

The amount of gas required to save data into storage is one of the highest among operations of the EVM. This cost is not always the same. Modifying a storage slot from a zero value to a non-zero one costs 20,000. While storing the same non-zero value or setting a non-zero value to zero costs 5,000. However, in the last scenario, when a non-zero value is set to zero, a refund of 15,000 will be given.

The EVM provides two opcodes to operate the storage:

* `SLOAD` loads a word from storage into the stack.
* `SSTORE` \*\*saves a word to storage.

These opcodes are also supported by the inline assembly of Solidity.

Solidity will automatically map every defined state variable of your contract to a slot in storage. The strategy is fairly simple — statically sized variables (everything except mappings and dynamic arrays) are laid out contiguously in storage starting from position 0.

For dynamic arrays, this slot (`p`) stores the length of the array and its data will be located at the slot number that results from hashing p(`keccak256(p)`). For mappings, this slot is unused and the value corresponding to a key `k` will be located at `keccak256(k,p)`. Bear in mind that the parameters of keccak256 (`k` and `p`) are always padded to 32 bytes.

#### Example

```solidity
pragma solidity ^0.5.11;

contract Storage {
  uint256 public number;            //storage[0]
  address public account;
  uint256[] private array;
  mapping(uint256 => uint256) private map;

  constructor() public {
    number = 2;
    account = address(this);
    array.push(10);
    array.push(100);
    map[1] = 9;
    map[2] = 10;
  }
}
```

storage\[0] → 2

storage\[1] → address of contract

storage\[2] → length of array

storage\[3] → unused, the mapping values are stored

**What is the Ethereum state and how is it modified?**

* Ethereum is a distributed state machine. Ethereum's state is a _machine state_, which can change from block to block according to a pre-defined set of rules, and which can execute arbitrary machine code.
* The specific rules of changing state from block to block are defined by the EVM.
* The state is an enormous data structure called a [modified Merkle Patricia Trie](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/), which keeps all [accounts](https://ethereum.org/en/developers/docs/accounts/) linked by hashes and reducible to a single root hash stored on the blockchain.

**How does the EVM work? (opcodes, bytecode, stacks etc.)**

* The EVM executes as a [stack machine](https://wikipedia.org/wiki/Stack\_machine) with a depth of 1024 items. Each item is a 256-bit word.
* During execution, the EVM maintains a transient _memory_, which does not persist between transactions.
* Contracts contain a Merkle Patricia _storage_ trie associated with the account in question and part of the global state.
* Compiled smart contract bytecode executes as a number of EVM [opcodes](https://ethereum.org/en/developers/docs/evm/opcodes), which perform standard stack operations like `XOR`, `AND`, `ADD`, `SUB`, etc. The EVM also implements a number of blockchain-specific stack operations, such as `ADDRESS`, `BALANCE`, `BLOCKHASH.`

<figure><img src="../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

**How many opcodes are there?**

141 opcodes, 1 byte each, 256 max

**What are the opcodes used for writing and querying memory and storage?**

* MLOAD and MSTORE - 3 gas
* SLOAD and SSTORE - 100 gas

**What is the maximum contract size?**

24,576 bytes (24KB)

**What are precompiled contracts and how do they work?**

* A special kind of contracts that are bundled with the EVM at fixed addresses, and can be called with a determined gas cost
* They are called from the opcodes like regular contracts, with instructions like [CALL](https://www.evm.codes/#F1)
* New hardforks may introduce new precompiled contracts
* Examples: `ecRecover, SHA2-256, identity, modexp, ecAdd, ecMul, ecPairing`
