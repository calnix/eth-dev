---
description: sload(slot) | sstore(slot, value) | (variable name).slot
---

# Storage Slots

* `y.slot` returns the storage slot assigned to variable y
* `sload(y.slot)` will load the value stored at the storage slot associated with variable y, specifically
* `sload(slot)`loads the value at the generically supplied slot number

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.17;

contract StorageBasics {
    uint256 x = 2;
    uint256 y = 13;
    uint256 z = 54;
    uint256 p;


    function getYstorage() external pure returns (uint256 ret) {
        assembly{
            ret := y.slot //returns storage slot of y variable (slot 1). not the value.
        }
    }

    // input slot number. returns the value stored in the slot as hexadecimal
    function getVarYul(uint256 slot) external view returns (bytes32 ret) {
        assembly {
            ret := sload(slot)  
        }
    }

    function setVarYul(uint256 slot, uint256 value) external {
        assembly {
            sstore(slot, value)
        }
    }
    
}
```

## Compacted storage variables

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.17;

contract StorageBasics {
    uint256 x = 2;
    uint128 a = 1;
    uint128 b = 2;


    function getASlot() external pure returns (uint256 ret) {
        assembly{
            ret := a.slot 
        }
    }
    
    function getBSlot() external pure returns (uint256 ret) {
        assembly{
            ret := b.slot 
        }
    }
    
    // input slot number. returns the value stored in the slot as hexadecimal
    function getVarYul(uint256 slot) external view returns (bytes32 ret) {
        assembly {
            ret := sload(slot)  
        }
    }    
}
```

* both functions will return the value of 1 -> a & b are both stored in storage slot 1
*   `getVarYul(1)`: the value of a & b are both returned together in a single hexadecimal numbera&#x20;

    <figure><img src="../../.gitbook/assets/image (322).png" alt=""><figcaption><p>a at the end, b at the middle</p></figcaption></figure>

### How do we load their values if there are in the same slot

* Storage offsets & Bit-shifting&#x20;
* all the variables are in slot 0

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.17;

contract StoragePart1 {
    uint128 public C = 4;
    uint96 public D = 6;
    uint16 public E = 8;
    uint8 public F = 1;


    function getOffsetE() external pure returns (uint256 slot, uint256 offset) {
        assembly {
            slot := E.slot
            offset := E.offset
        }
    } 
    
    function readBySlot(uint256 slot) external view returns (bytes32 value) {
        assembly {
            value := sload(slot)
        }
    }
}
```

* variable E is compacted into slot 0, with C and D and F
* offset is 28: tells us exactly where to look for to find the variable
  * you look 28 bytes to the left, starting from the end, you will find the variable.

<figure><img src="../../.gitbook/assets/image (215).png" alt=""><figcaption></figcaption></figure>

**slot 0 value:** \
`0x00010008`<mark style="color:red;">`00000000000000000000000600000000000000000000000000000004`</mark>

* 64  hex characters -> 64/2 = 32 bytes (2 hex char is 1 byte)

<mark style="color:red;">`00000000000000000000000600000000000000000000000000000004`</mark>

* 56 hex characters -> 56/2 = **28 bytes**

{% hint style="info" %}
if we want to read `E` , take the value in slot 0 and shift it to the right by 28 bytes.
{% endhint %}

Let's see the code to shift right:

```solidity
function readE() external view returns (uint256 e) {
        assembly {
            let value := sload(E.slot) // must load in 32 byte increments
            //
            // E.offset = 28 bytes
            let shifted := shr(mul(E.offset, 8), value)
            // 0x0000000000000000000000000000000000000000000000000000000000010008
            // equivalent to
            // 0x000000000000000000000000000000000000000000000000000000000000ffff
            e := and(0xffff, shifted)
}
```

* `shr(bits, value)`: shifts the bits in the `value`, rightward, by `bits`
* shift right takes bits as an argument, not bytes. so we must multiply the offset by 8 to get bits.
* ```
              let shifted := shr(mul(E.offset, 8), value)
  ```

#### **right-shift:**

* `0x00010008`<mark style="color:red;">`00000000000000000000000600000000000000000000000000000004`</mark>
* <mark style="color:red;">`to`</mark>
* `0x`<mark style="color:red;">`00000000000000000000000000000000000000000000000000000000`</mark>`00010008`
* everything in red on the right drops off. the value moves rightward to the end. leading 0s infront

{% hint style="info" %}
Note that we still have the 1 to the left of 8, which is the value of F
{% endhint %}

#### To remove the 1:

```solidity
// 0x0000000000000000000000000000000000000000000000000000000000010008
// equivalent to
// 0x000000000000000000000000000000000000000000000000000000000000ffff
e := and(0xffff, shifted)
```

* and() will returns a 1 in each bit position for which the corresponding bits of both operands are 1
* so the leading zeros will remain 0
* since we wanna drop the 1, to is matched with a 0
* to keep the rest, we use f - which is 1111&#x20;
  * bin(0xf) = 1111

## TBC

