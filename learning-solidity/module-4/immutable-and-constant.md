# Immutable and Constant

* Using the `constant` and the `immutable` keywords for variables that do not change helps to save on gas used.&#x20;
* `constant` and `immutable` variables **do not occupy a storage slot** when compiled.&#x20;
* They are saved inside the contract byte code.&#x20;

### What is a constant variable? <a href="#9286" id="9286"></a>

A constant variable in Solidity is a variable whose value cannot be changed once set. This means that a constant variable can only be assigned a value once, and it cannot be reassigned or modified thereafter.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract Constants {
    address public constant myAddress = 0x777788889999AaAAbBbbCcccddDdeeeEfFFfCcCc;
    uint public constant myUint = 143;
}
```

* assignment has to be done at declaration

### What is an immutable variable? <a href="#b8a9" id="b8a9"></a>

An immutable variable in Solidity refers to a variable whose reference cannot be changed. In other words, an immutable variable is a variable that points to an object, and the object’s value cannot be changed, but the variable can be reassigned to point to a new object.

> For example, if you have an immutable variable named “y” that points to a string “Hello”, you cannot change the string to “Goodbye”, but you can reassign “y” to point to a new string “Hola”.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract Immutable {
    address public immutable myAddress;
    uint public immutable myUint;

    constructor(uint _myUint) {
        myAddress = msg.sender;
        myUint = _myUint;
    }
}
```

* Values of immutable variables **can be set inside the constructor but cannot be modified afterward.**&#x20;

{% hint style="info" %}
Constants are typically used when you want to ensure that a value cannot be changed, while immutables are used when you want to ensure that a reference cannot be changed.
{% endhint %}
