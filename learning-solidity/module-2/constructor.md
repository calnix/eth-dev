# Constructor

A constructor is a special type of function that is automatically called when the contract is created. Constructors are typically used to set up initial state variables and any other setup that needs to happen when the contract is first deployed.

* The code defined inside the constructor will run only once, at the time the contract is created.

```solidity
contract SimpleConstructor {    
    uint public x; 
    
    constructor(uint _x) public {        
        x = _x;    
    }
}
```

* In this example, the constructor takes in a uint and sets it to the contract's x state variable.

### Deployment

After the constructor has executed, the final bytecode of the contract is stored on the blockchain.

* The deployed bytecode does not contain the constructor code, or internal functions only called from the constructor.
* The bytecode includes all public and external functions and all functions that are reachable from there through function calls.&#x20;

{% hint style="info" %}
[OpenZeppelin explains the idea more precisely in the second part of their article series “Deconstructing a Solidity Contract”](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-ii-creation-vs-runtime-6b9d60ecb44c/):

The constructor code is part of the creation code, not part of the runtime code.
{% endhint %}

### Inheritance

If a base contract has arguments, derived contracts need to specify all of them. This can be done in **two** ways:

#### **1.  Directly in the inheritance declaration**

```solidity
pragma solidity 0.8.0;

contract Animal {
    
    uint feet;
    bool canSwim;
    
    constructor(uint _feet, bool _canSwim) {
        feet = _feet;
        canSwim = _canSwim;
    }
}

// inherit and set constructor params
contract Lion is Animal(4, true) {
}
```

#### **2.  Through the derived constructor**

```solidity
contract Animal {
    
    string name;
    uint feet;
    bool canSwim;
    
    constructor(string memory _name, uint _feet, bool _canSwim) {
        name = _name;
        feet = _feet;
        canSwim = _canSwim;
    }
}
contract Lion is Animal {
    
    constructor(string memory _name) Animal(_name, 4, true) {
        // ...
    }
}
```

This way can be used if the constructor arguments of the base contract depend on those of the derived contract.

## Payable Constructor <a href="#882b" id="882b"></a>

* Constructors can accept Ether. In this case, they should be declared with the keyword “payable”.&#x20;
* If you try to add ether when deploying a contract defined with a non payable constructor, it will throw you an exception and revert.
* The difference between a payable and non payable constructor is also reflected in the creation code.
* If the constructor is non-payable, the creation code contains 8 low-levels instructions (in EVM assembly), that check if some ethers has been sent to the contract once it has been deployed. If it is the case, the check fails and the contract reverts.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract Payable {
    // Payable address can receive Ether
    address payable public owner;

    // Payable constructor can receive Ether
    constructor() payable {
        owner = payable(msg.sender);
    }

 // .... rest of code .... //
}
```
