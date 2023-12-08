# Memory: how solidity uses memory

## How solidity uses memory

* scratch space: \[0x00 - 0x20), \[0x20 - 0x40)
* free memory pointer: \[0x40 - 0x60)
* empty: \[0x60 - 0x80)
* action begins from \[0x80 - ...

### scratch space

* write values here and expect them to be epheremal&#x20;
* previous operations may have left over values here -> space is not guaranteed to be clear
* if you are writing in 32 bytes increments then you will overwrite anything present

### free memory pointer

* points to the closest empty memory slot
* since solidity does not garbage collect/free memory, the pointer will never decrement
* initially points towards **0x80**

```solidity
contract Memory {
    struct Point {
        uint256 x;
        uint256 y;
    }

    event MemoryPointer(bytes32);
    event MemoryPointerMsize(bytes32, bytes32);

    function memPointer() external {
        bytes32 x40;
        assembly {
            x40 := mload(0x40)
        }
        emit MemoryPointer(x40);
        Point memory p = Point({x: 1, y: 2});

        assembly {
            x40 := mload(0x40)
        }
        emit MemoryPointer(x40);
    }
}
```

memPointer emits 0x80 first, then 0xc0

<figure><img src="../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

64 bytes, corresponding to the struct that consists of two 32 bytes (x & y in struct)

### Arrays

memory arrays have no `push`, unlike dynamic storage arrays.

* because objects in memory are laid out end to end, pushing might result in collision with another variable

In Yul, you can read a variable declared as memory, it will reference the variable's memory location - not its actual value. See `location` below:

```solidity
    event Debug(bytes32, bytes32, bytes32, bytes32);

    function args(uint256[] memory arr) external {
        bytes32 location;
        bytes32 len;
        bytes32 valueAtIndex0;
        bytes32 valueAtIndex1;
        assembly {
            location := arr
            len := mload(arr)
            valueAtIndex0 := mload(add(arr, 0x20))
            valueAtIndex1 := mload(add(arr, 0x40))
            // ...
        }
        emit Debug(location, len, valueAtIndex0, valueAtIndex1);
    }
```

* arr is a dynamic array, in memory
* reading arr returns its memory location -> stored in location
* first 32 bytes is length
* 32 bytes after is first element

{% hint style="info" %}
Dynamic array will begin with how long the array is. To access first elemtn, you have to add 32bytes or 0x20 ot skip length
{% endhint %}

