# Mapping

Mapping is a data structure in Solidity that allows a programmer to store, retrieve and modify data in a key-value pair format. It is similar to a hash table or dictionary in other languages.

**Example:**

Let's assume we want to store a list of people and their ages. We can do this with a mapping like this:

```solidity
mapping (address => uint) public peopleAges;
```

## Initialization & Assignment

* On Initialization, all possible key-value pairs are already initialized.&#x20;
* Accessing unassigned key-value pairs will return the default value for that value type.
* This is because the hash that points to the location in storage has not been generated & we have not written a value thus far.

#### Assignment of values must be done inside a function

* cannot simply do `myMapping[1] = "test"`  (like python)

**Example**

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract SimpleMappingExample {

    mapping(uint => bool) public myMapping;
    mapping(address => bool) public myAddressMapping;

    function setValue(uint _index) public {
        myMapping[_index] = true;
    }

    function setMyAddressToTrue() public {
        myAddressMapping[msg.sender] = true;
    }
}
```

![](<../../.gitbook/assets/image (110).png>)         ![](<../../.gitbook/assets/image (7).png>)

1. On initialization, `myMapping[1]` will return the default bool value of false.&#x20;
2. Then we call `setValue` setting -> `myMapping[1]` = true
3. Subsequently, when we call `myMapping[1]`, we are returned true as set earlier.

### Nested Mapping

```solidity
mapping(address => mapping(uint256 => string)) public userData;

// accessing nested mapping
userData[address][uint256] = 'John'
```

* This creates a mapping from an address to a **mapping of uints to strings.**&#x20;
* This allows us to **store a string value associated with a given address and a given uint**.&#x20;
* This could be used to store user data associated with a particular address and a particular event or transaction.
