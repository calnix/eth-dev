---
description: '// TODO: Read the state variable "e" using only assembly'
---

# read state variable

{% tabs %}
{% tab title="correct" %}
```solidity
contract Question2 {
    address private a;
    uint64 private b;
    bool private c;
    address private d;
    uint256 private e = 55;
    uint256 private f;
    address private g;

    function readStateVariable() external view returns (uint256) {
        // TODO: Read the state variable "e" using only assembly
        uint256 result;

        assembly {
        // Load the value of the storage slot where "e" is stored
            result := sload(e.slot)
        }
        return result;
    }
}
```
{% endtab %}

{% tab title="wrong" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract Question2 {
    address private a;
    uint64 private b;
    bool private c;
    address private d;
    uint256 private e = 55;
    uint256 private f;
    address private g;

    function readStateVariable() external view returns (uint256) {
        // TODO: Read the state variable "e" using only assembly
        assembly {
        // Load the value of the storage slot where "e" is stored
            result := sload(e.slot)
            mstore(0x0, result)    // store result in memory
            return(0x0, 32)       // return 32 bytes from memory
        }

    }
}
```

{% hint style="info" %}
The free memory pointer (stored at 0x40) starts at 0x80 simply because there are 4 32 byte slots at the start of memory that are reserved. From the Solidity docs on the [memory layout](https://docs.soliditylang.org/en/v0.8.17/internals/layout\_in\_memory.html):

* 0x00 - 0x3f (64 bytes): scratch space for hashing methods
* 0x40 - 0x5f (32 bytes): currently allocated memory size (aka. free memory pointer)
* 0x60 - 0x7f (32 bytes): zero slot

If the free memory pointer started any earlier than 0x80, it would interfere with these reserved slots (including the free memory pointer itself)
{% endhint %}
{% endtab %}
{% endtabs %}

```
/** Explanation:

Storage is set up based on 32 byte words.
Variables that are smaller than 32 bytes will be stored in the same word.
Multiple, contiguous items that need less than 32 bytes are packed into a single storage slot if possible.

storage slot 0 -> a,b,c 
- has 32-29 = 3 bytes free

storage slot 1 -> d
- has 32-20 = 12 bytes free
- d has to occupy a new slot since it cannot fit into slot 0.

storage slot 2 -> e
- uint256 variable occupies an entire word. 
- e has to occupy a new slot.

sload(e.slot) will load the value stored at the storage slot associated with variable e, specifically.
This is loaded into memory via mstore(0x0, result).

We opt to load into the scratch space, as scratch space can be used between statements (i.e. within inline assembly). 
There is no concern of spillage out of scratch space, as we are loading 32 bytes into a 64 byte scratch space.

Finally, we return the value stored at 0x00; returning 32 bytes reflecting what was initially loaded. 

*/
```
