# Variable Scope: State & Local variables

## **Variable Scope**

**Storage variables (aka state variables)**

* Stored in the blockchain and their values are persistent and publicly accessible.&#x20;
* Declared outside a function.

```solidity
contract MyContract {    
    uint public balance; // Declares a state variable
}
```

**Local variables**

* Declared within a function
* Not stored on the blockchain

```solidity
contract MyContract {    

    function addOne() public {        
        uint256 localNumber = 2;        
    }
}
```

{% hint style="info" %}
Local variables are useful for storing temporary values or intermediate results and can be declared in the same way as any other type of variable.
{% endhint %}

## Contract layout

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Variables {
    
    // State variables are stored on the blockchain.
    string public text = "Hello";
    uint public num = 123;

    function doSomething() public {
        // Local variables are not saved to the blockchain.
        uint i = 456;

        // Here are some global variables
        uint timestamp = block.timestamp;  // Current block timestamp
        address sender = msg.sender;       // address of the caller
    }
}
```

### Visibility&#x20;

State variables only have three possible visibility modifiers:&#x20;

1. **public** (accessed internally as well as via external function calls)
2. **internal** (only accessed internally and inheriting contracts)
3. **private**  (only within the contract it is defined)

When visibility is not specified, state variables have the default value of **internal**.&#x20;

## Updating State Variables

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract SimpleStorage {
    // State variable to store a number
    uint public num;

    // You need to send a transaction to write to a state variable.
    function set(uint _num) public {
        num = _num;
    }

    // You can read from a state variable without sending a transaction.
    function get() public view returns (uint) {
        return num;
    }
}
```

* To write or update a state variable you need to send a transaction.&#x20;
* On the other hand, you can read state variables for free, as it does not involve a transaction.

{% hint style="warning" %}
If you have **public** _state variables_ in your contract, the compiler will create _getter_ functions for these automatically. \
\--> <mark style="background-color:orange;">`get()`</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">is not required</mark>
{% endhint %}
