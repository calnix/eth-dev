# Storage of Arrays and Mappings

### Fixed array

* behaves just like typical value type variables
* Structs and array data always start a new slot and their items are packed tightly according to these rules.

```solidity
contract StorageComplex {
    uint256 a;               //slot 0
    uint256[3] fixedArray;   //slot 1,2,3: 99,999,9999
    uint256 b;               //slot 4
    
     constructor() {
        fixedArray = [99, 999, 9999];
    }
    
    function getArraySlot() external pure returns(uint256 ret) {
        assembly {
            ret := fixedArray.slot  //returns 1
        }
    }

    function getVarSlot() external pure returns(uint256 ret) {
        assembly {
            ret := b.slot  //returns 4
        }
    }

    // to get individual elements
    function fixedArrayView(uint256 index) external view returns (uint256 ret) {
        assembly {
            ret := sload(add(fixedArray.slot, index))
        }
    }
}
```

* `getArraySlot()` -> returns 1 -> storage slot 1
  * returns the first slot the array occupies
  * `fixedArray` occupies storage slot 1 to 3: 1 slot for each of its uint256 elements
* `fixedArrayView()` -> index: {0,2}
  * index(0): 99
  * index(1): 999
  * basically we are incrementing the slot by the index, to move to the appropriate storage slot containing the element

### Dynamic array

* elements will not be stored sequentially down the slots, like a fixed array.
* cos' it could expand, overrun and crash into something below it.
* length of the dynamic array is stored at its slot -> `sload(bigArray.slot)`
* Array data is located starting at `keccak256(array.slot)`

```solidity
contract StorageComplex2 {
    uint256[3] fixedArray;  //slot: 0,1,2
    uint256[] bigArray;     //slot: 3 
    uint8[] smallArray;

  constructor() {
        fixedArray = [99, 999, 9999];  
        bigArray = [10, 20, 30, 40];   
        smallArray = [1, 2, 3];
    }
    
    // returns slot of big array
    function getBigArraySlot() external pure returns(uint256 ret) {
        assembly {
            ret := bigArray.slot  //returns 3
        }
    }
    
    //returns length of dynamic array
    function bigArrayLength() external view returns (uint256 ret) {
        assembly {
            ret := sload(bigArray.slot) //returns 4
        }
    }
    
    // get elements in big array
    function readBigArrayLocation(uint256 index) external view returns (uint256 ret) {
        uint256 slot;

        assembly {
            slot := bigArray.slot
        }
        bytes32 location = keccak256(abi.encode(slot));

        assembly {
            ret := sload(add(location, index))
        }
    }
}
```

* elements are stored at different storage location: keccak256 hash of the slot
* to read elements sequentially, add indexed to the location to traverse down the list of elements

{% hint style="info" %}
When you take the hash of a number, its gonna land in this enormous 256bit space. Chances are it is unlikely that the hash will collide/crash into the hash of another different number.

Thus solidity takes the approach of hashing to be able to grow arrays freely, and not crash into any other storage slots.
{% endhint %}

### uint8\[] smallArray

```solidity
contract StorageComplex2 {
    uint256[3] fixedArray;  //slot: 0,1,2
    uint256[] bigArray;     //slot: 3 
    uint8[] smallArray;

  constructor() {
        fixedArray = [99, 999, 9999];  
        bigArray = [10, 20, 30, 40];   
        smallArray = [1, 2, 3];
    }

    // returns length of array: 3
    function readSmallArray() external view returns (uint256 ret) {
        assembly {
            ret := sload(smallArray.slot)
        }
    }

    // returns elements in array
    function readSmallArrayLocation(uint256 index) external view returns (bytes32 ret) {
        uint256 slot;
        assembly {
            slot := smallArray.slot
        }
        bytes32 location = keccak256(abi.encode(slot));

        assembly {
            ret := sload(add(location, index))
        }
    }
}
```

* since uint8 is 1 bytes, all 3 elements are packed into the first storage slot at location
* all elements stored at index 0, nothing at index 1.

<figure><img src="../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

### Mappings

* like dynamic arrays location of elements are stored elswhere
* location: hash of the concatenation the key with the storage slot
* ```
  bytes32 location = keccak256(abi.encode(key, uint256(slot)));
  ```

{% hint style="info" %}
For mappings, the slot stays empty, but it is still needed to ensure that even if there are two mappings next to each other, their content ends up at different storage locations.
{% endhint %}

```solidity
contract StorageComplex2 {

    mapping(uint256 => uint256) public myMapping;
    mapping(uint256 => mapping(uint256 => uint256)) public nestedMapping;
    mapping(address => uint256[]) public addressToList;

    constructor() {
        myMapping[10] = 5;
        myMapping[11] = 6;
        nestedMapping[2][4] = 7;

        addressToList[0x5B38Da6a701c568545dCfcB03FcB875f56beddC4] = [
            42,
            1337,
            777
        ];
    }
    
    // returns values in a mapping
    function getMapping(uint256 key) external view returns (uint256 ret) {
        uint256 slot;
        assembly {
            slot := myMapping.slot
        }

        bytes32 location = keccak256(abi.encode(key, uint256(slot)));

        assembly {
            ret := sload(location)
        }
    }
    
    // nestedMapping[2][4] = 7;
    function getNestedMapping() external view returns (uint256 ret) {
        uint256 slot;
        assembly {
            slot := nestedMapping.slot
        }

        bytes32 location = keccak256(abi.encode(uint256(4),
                keccak256(abi.encode(uint256(2), uint256(slot)))
            )
        );
        assembly {
            ret := sload(location)
        }
    }

    // mapping(address => uint256[]) public addressToList;
    function lengthOfNestedList() external view returns (uint256 ret) {
        uint256 addressToListSlot;
        assembly {
            addressToListSlot := addressToList.slot
        }

        bytes32 location = keccak256(
            abi.encode(
                address(0x5B38Da6a701c568545dCfcB03FcB875f56beddC4),
                uint256(addressToListSlot)
            )
        );
        assembly {
            ret := sload(location)
        }
    }

    function getAddressToList(uint256 index) external view returns (uint256 ret) {
        uint256 slot;
        assembly {
            slot := addressToList.slot
        }

        bytes32 location = keccak256(
            abi.encode(
                keccak256(
                    abi.encode(
                        address(0x5B38Da6a701c568545dCfcB03FcB875f56beddC4),
                        uint256(slot)
                    )
                )
            )
        );
        assembly {
            ret := sload(add(location, index))
        }
    }

}
```
