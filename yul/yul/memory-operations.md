# Memory Operations

**Equivalent to heap in other languages**

* but there is no garbage collector of `free`
* memory is laid out in 32 bytes sequences
* \[0x00 - 0x20) \[0x20 - 0x40) \[0x40 - 0x60)...

**Only four instructions**

* `mload`, `mstore`, `mstore8`, `msize`
* `mstore(p, v)` stores value v in slot p (_like sload_)
* `mload(p)` retrieves 32 bytes from slot p  \[p - 0x02]
* `mstore8(p, v)` like mstore, but for 1 byte
* `msize()` largest accessed memory index in that transaction

{% hint style="info" %}
In pure yul contracts, memory is easy to use -> can be treated as an array

But in mixed solidity/yul contracts, solidity expects memory to be used in a specific manner.
{% endhint %}

### Gas

* you are charged gas for each memory access, and for **how far into the memory array you accessed**
* **mload(0xffffffffffffffff) will run out of gas**
  * only have 30M gas in a block, and the mload will expend more than all of it
* **using a hash function to mstore like storage does is a bad idea**

### Packing/Unpacking

* EVM memory does not pack datatypes smaller than 32 bytes
* If you load from storage to memory, it will be unpacked&#x20;
  * ergo: can pack storage vars, but not when moving them memory

```solidity
uint8[] foo = [1,2,3,4,5,6];     //32 bytes

function unpacked() external {
    uint8[] memory bar = foo;
}
```

* foo will occupy 1 storage slot
* when we run unpacked(), each element will occupy its own 32 byte slot in memory.
