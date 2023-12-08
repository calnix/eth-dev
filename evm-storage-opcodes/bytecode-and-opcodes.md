# ByteCode and Opcodes

## Solidity → Bytecode → Opcode

Solidity code is compiled into bytecode prior to being deployed to the Ethereum network. This bytecode **corresponds to a series of opcode instructions** that the EVM interprets.

The runtime bytecode created from Solidity contracts is a representation of the entire contract. Within the contract, you may have multiple functions that can be called once it is deployed.

For example Storage.sol: contract has 2 functions, store(uint256) and retrieve.

<figure><img src="../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

Below is the compiled runtime bytecode of the entire contract:

{% code title="Storage.sol bytecode" overflow="wrap" %}
```
608060405234801561001057600080fd5b50600436106100365760003560e01c80632e64cec11461003b5780636057361d14610059575b600080fd5b610043610075565b60405161005091906100d9565b60405180910390f35b610073600480360381019061006e919061009d565b61007e565b005b60008054905090565b8060008190555050565b60008135905061009781610103565b92915050565b6000602082840312156100b3576100b26100fe565b5b60006100c184828501610088565b91505092915050565b6100d3816100f4565b82525050565b60006020820190506100ee60008301846100ca565b92915050565b6000819050919050565b600080fd5b61010c816100f4565b811461011757600080fd5b5056fea2646970667358221220404e37f487a89a932dca5e77faaf6ca2de3b991f93d230604b1b8daaef64766264736f6c63430008070033
```
{% endcode %}

This code includes all **public and external** functions and all functions that are reachable through function calls. The deployed code does not include the constructor code or internal functions only called from the constructor.

* In short, the bytecode would contain the functions hash/signatures.

We are going to focus on the snippet of bytecode below. This snippet represents the function selector logic. Run “ctrl f” on the snippet to verify it is in the above bytecode.

```
60003560e01c80632e64cec11461003b5780636057361d1461005957
```

This bytecode corresponds to a set of EVM opcodes and their input values.

The below shows the bytecode snippet broken into its corresponding opcode commands. These are run sequentially on the call stack by the EVM.

```
60 00                       =   PUSH1 0x00
35                          =   CALLDATALOAD
60 e0                       =   PUSH1 0xe0
1c                          =   SHR
80                          =   DUP1
63 2e64cec1                 =   PUSH4 0x2e64cec1
14                          =   EQ
61 003b                     =   PUSH2 0x003b
57                          =   JUMPI
80                          =   DUP1
63 6057361d                 =   PUSH4 0x6057361d
14                          =   EQ
61 0059                     =   PUSH2 0x0059
57                          =   JUMPI
```

{% hint style="success" %}
Opcodes are 1 byte in length leading to 256 different possible opcodes. The EVM only uses 140 unique opcodes. You can check out the list of EVM opcodes [here](https://www.ethervm.io/)
{% endhint %}

{% hint style="info" %}
The function hash is calculated using the first 4 bytes of the keccak256 hash of the function signature.

Function signature, if no parameters:

> retrieve() first 4 bytes of keccak256 hash: bytes4(keccak256("retrieve()");

If the function has input parameters:

> store(uint256 num) bytes4(keccak256(”store(uint256 num)”));
{% endhint %}

### Smart Contract Function Calls & Calldata

When we call a contract function we include some calldata that specifies the function signature we are calling and any arguments that need to be passed in.

calldata → function signature + any required parameters

<figure><img src="../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

* Here we are making a contract call to the store function with argument 10.
* Use `abi.encodeWithSignature()` to get the calldata in the desired format.
* The emit logs our calldata for testing.
* functionCalldata = `0x6057361d000000000000000000000000000000000000000000000000000000000000000a`

The above is what abi.encodeWithSignature("store(uint256)", 10) returns.

```solidity
Function signatures:

keccak256(“store(uint256)”) →  first 4 bytes = 6057361d
keccak256(“retrieve()”) → first 4 bytes = 2e64cec1
```

Looking at our calldata above we can see that we have **36 bytes of calldata,**

* first 4 bytes of our calldata correspond to the function selector we just computed for the store(uint256) function.
* remaining 32 bytes correspond to our uint256 input argument.
* we have a hex value of “a” which is equal to 10 in decimal

{% code overflow="wrap" %}
```
6057361d = function signature (4 bytes)

000000000000000000000000000000000000000000000000000000000000000a = uint256 input (32 bytes)
```
{% endcode %}

If we take the function signature 6057361d and refer back to the opcode section, run “ctrl f” on this value and see if you can find it.

### Opcodes & The Call Stack

EVM is a stack-based machine, and thus performs all computations in a data area called the stack. All in-memory values are also stored in the stack.

<figure><img src="../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

Stack is LIFO: last-in, first-out.

* Push: Add item to top
* Pop: remove item from top
* For more info:[https://www.youtube.com/watch?v=FNZ5o9S9prU](https://www.youtube.com/watch?v=FNZ5o9S9prU)

**Stack has a maximum depth of 1024 words**&#x20;

* can hold 1024 things
* stack\[0] is the value at the top of the stack
* stack\[1] is the value one below the top of the stack.
* 1 word is 256 bits or 32 bytes.

<figure><img src="../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

**PUSH1 0x80:**

* Push 1 byte onto the stack (the 1 byte here is 0x80)
* So when you push 1 byte onto the stack, the top 31 bytes are set to 0

{% hint style="info" %}
calldata is 36 bytes: 4bytes (function selector) + 32 bytes (parameters/else padded 0s) \
\
**CALLDATALOAD** \
pops off the first value on the stack as input. This opcode loads in the calldata to the stack using the input value as an offset. \
\
Why is this necessary? calldata = 36 bytes, stack = 32 bytes. cannot fit the entire calldata into stack. \
\
So input dictates which 32 bytes is loaded onto stack. The pushed value is [msg.dat](http://msg.data/)a\[i :i+32] Assuming 0 (0x00) was at stack\[0]. It would be used as input, therefore, i =0, and the first 32 bytes of calldata would be loaded. \
\
This would exclude the last 4 bytes containing parameters, if any. Of `0x6057361d000000000000000000000000000000000000000000000000000000000000000a` the trailing `0000000a`
{% endhint %}

## Memory

Contract memory is a simple byte array, where data can be stored in 32 bytes (256 bit) or 1 byte (8 bit) chunks and read in 32 bytes (256 bit) chunks.

source: [https://takenobu-hs.github.io/downloads/ethereum\_evm\_illustrated.pdf](https://takenobu-hs.github.io/downloads/ethereum\_evm\_illustrated.pdf)

<figure><img src="../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

This functionality is determined by the 3 opcodes that operate on memory.

* MSTORE (x, y) - Store a 32 byte (256-bit) value “y” starting at memory location “x”
* MLOAD (x) - Load 32 bytes (256-bit) starting at memory location “x” onto the call stack
* MSTORE8 (x, y) - Store a 1 byte (8-bit) value “y” at memory location “x” (the least significant byte of the 32-byte stack value).

{% hint style="info" %}
This [EVM playground](https://www.evm.codes/playground?unit=Wei\&codeType=Mnemonic\&code=%27Vg\*\(\_I...1W0GJ\_!!!!z00FK22WJQ0Y22z20F8K33W33Q1Y33z21F8d\(v0Z0-Jq00Xd\(vJZJ-64q20Xdv33Z33-65q21Xpp%27\~N%20locatioCzG1\_wppVv7o7hBcall%20stack%20from\~uIIIIq\(%20ofNzp%5Cnj%20bytegSTOREdw\)\*\_%200xZ9BY9Chex%7DzXpM\)W%20at\~V%2F%2F%20MQ%20%7B0x2N%20memoryKwg8%201j\_J32I11GpPUSHFpMgCn%20Be%209%20i7%20t\*%20J\)LOAD\(js!uu%01!\(\)\*79BCFGIJKNQVWXYZ\_dgjpquvwz\~\_) will help solidify your understanding of what these 3 opcodes Remember: 8 bits → 1 byte 2 hex characters → 1 byte (1 hex char = 4 bits)
{% endhint %}

#### EVM playground walkthrough

```solidity
// MSTORE 32 bytes 0x11...1 at memory location 0
PUSH32 0x1111111111111111111111111111111111111111111111111111111111111111
PUSH1 0x00  // == 0 (in base 10)
MSTORE
```

<figure><img src="../.gitbook/assets/image (163).png" alt=""><figcaption><p>before MSTORE</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (124).png" alt=""><figcaption><p>after MSTORE</p></figcaption></figure>

Now memory has been filled from index 0 to 31 (32 bytes occupied). Next:

```solidity
// MSTORE8 1 byte 0x22 at memory location 32 (0x20 in hex)
PUSH1 0x22
PUSH1 0x20   // == 32 (in base 10)
MSTORE8
```

<figure><img src="../.gitbook/assets/image (144).png" alt=""><figcaption><p>before MSTORE</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (193).png" alt=""><figcaption><p>after MSTORE</p></figcaption></figure>

Since we only want to store 1 byte (0x22), we see 22 appended to memory, from index 32 to 33. However, padding of 0s are added to occupy the full 32 bytes (256 bits).

Note that all locations in memory are well-defined initially as zero which is why we see 2200000000000000000000000000000000000000000000000000000000000000 added to our memory.

<mark style="color:blue;">**This is because:**</mark> our memory was 32 bytes before we wrote 1 byte at location 32. At this point we began writing into untouched memory, as a result, the memory was expanded by another 32-byte increment to 64 bytes.

{% hint style="info" %}
**Memory Expansion** \
When your contract writes to memory, you have to pay for the number of bytes written. If you are writing to an area of memory that hasn't been written to before, there is an additional memory expansion cost for using it for the first time.

Memory is expanded in 32 bytes (256-bit) increments when writing to previously untouched memory space.
{% endhint %}

```solidity
// MSTORE8 1 byte 0x33 at memory location 33 (0x21 in hex)
PUSH1 0x33
PUSH1 0x21
MSTORE8
```

<figure><img src="../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

On adding another singe byte to memory, it just overwrites the padded zeroes.

#### MLOAD

```
// MLOAD 32 bytes to the call stack from memory location 0 ie 0-32 bytes of memory
PUSH1 0x00
MLOAD
```

```solidity
// MLOAD 32 bytes to the call stack from memory location 32 ie 32-64 bytes of memory
PUSH1 0x20
MLOAD
```

```solidity
// MLOAD 32 bytes to the call stack from memory location 33 ie 33-65 bytes of memory
PUSH1 0x21
MLOAD
```

<figure><img src="../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

### Remember Memory is a Byte Array

The second thing you may have noticed occurred when we ran an MLOAD from memory location 33 (0x21). We returned the following value to the call stack.

```
3300000000000000000000000000000000000000000000000000000000000000
```

**We were able to start our read from a non 32 factor.**

Remember memory is a byte array meaning we can start our reads (and our writes) from any memory location. We are not constrained to multiples of 32. Memory is linear and can be addressed at the byte level.

> Memory can only be newly created in a function. It can either be newly instantiated complex types like array/struct (e.g. via new int\[...]) or copied from a storage referenced variable.

### Free memory pointer

* is simply a pointer to the location where free memory starts.
* ensures smart contracts keep track of which memory locations have been written to and which haven’t.
* protects against a contract overwriting some memory that has been allocated to another variable.
* When a variable is written to memory the contract will first reference the free memory pointer to determine where the data should be stored.
* It then updates the free memory pointer by noting how much data is to be written to the new location. A simple addition of these 2 values will yield where the new free memory will start.
* `freeMemoryPointer + dataSizeBytes = newFreeMemoryPointer`

As mentioned before the free memory pointer is defined at the start of the runtime bytecode through these 5 opcodes.

```
60 80                       =   PUSH1 0x80
60 40                       =   PUSH1 0x40
52                          =   MSTORE
```

These effectively state that the free memory pointer is

* located in memory at byte 0x40 (64 in decimal)
* has a value of 0x80 (128 in decimal).

The immediate questions you may have are why the values 0x40 & 0x80 are used above. The answer to this:

> Solidity’s memory layout reserves four 32-byte slots (4\*32 = **128**):
>
> * `0x00` - `0x3f` (64 bytes): scratch space
> * `0x40` - `0x5f` (32 bytes): free memory pointer
> * `0x60` - `0x7f` (32 bytes): zero slot
> * width of reserved space = 32 + 32 +64 = **128 bytes**

We can see that 0x40 is the predefined location by solidity for the free memory pointer. The value 0x80 is merely the first memory byte that is available to write to **after the 4 reserved 32-byte slots.**

**Reserved section:**

* Scratch space, can be used between statements i.e. within inline assembly and for hashing methods.
* Free memory pointer, currently allocated memory size, start location of free memory, 0x80 initially.
* The zero slot, is used as an initial value for dynamic memory arrays and should never be written to.

{% hint style="info" %}
[https://noxx.substack.com/p/evm-deep-dives-the-path-to-shadowy-d6b?s=r](https://noxx.substack.com/p/evm-deep-dives-the-path-to-shadowy-d6b?s=r)
{% endhint %}

### Example: Memory in a Real Contract

<figure><img src="../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

* single function that defines two arrays of lengths 5 & 2 and then assigns b\[0] a value of 1.
* [evm playground](https://rb.gy/amswxm) for MemoryLane.sol

#### Free Memory Pointer Initialisation (EVM Playground Lines 1-15)

[evm playground](https://tinyurl.com/mtem6fzu), Lines 1-15:

1. `0x80` (**128**) is pushed onto stack → value of free memory pointer
   1. free memory starts after 128 bytes
2. `0x40` is pushed onto stack → location of free memory point
   1. location tells us where to get the value for free memory pointer
      1. **`MSTORE`**
         1. pops the first item off the stack `0x40` to determine **where** to write to in memory
         2. the second value `0x80` as **what** to write

<figure><img src="../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

* Stack is empty, but we have populated some memory.
* This memory representation is in hexadecimal where each character represents 4 bits.
* We have 192 hexadecimal characters in memory which means we have 96 bytes (1 byte = 8 bits = 2 hexadecimal characters).

If we refer back to Solidity’s memory layout we were told the first 64 bytes would be allocated as scratch space and the next 32 would be for the free memory pointer.

{% hint style="danger" %}
Of course, we have not yet initialised the last reserved section: zero slot.
{% endhint %}



