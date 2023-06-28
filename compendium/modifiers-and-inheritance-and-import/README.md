# Modifiers & Inheritance & Import

### Modifiers&#x20;

are a great way to check pre-conditions.

```solidity
//SPDX-License-Identifier: MIT

pragma solidity 0.8.3;

contract ModifierExample {

    mapping(address => uint) public tokenBalance;
    address owner;
    uint tokenPrice = 1 ether;

    constructor() {
        owner = msg.sender;
        tokenBalance[owner] = 100;
    }

    function createNewToken() public {
        require(msg.sender == owner, "You are not the owner");
        tokenBalance[owner]++;
    }

    function burnToken() public {
        require(msg.sender == owner, "you are not allowed");
        tokenBalance[owner]--;
    }
}
```

Both functions burnToken() and createNewToken() have the same require statement. We can centralize this by using a modifier instead.

```solidity
    modifier onlyOwner() {
        require(msg.sender == owner, "You are not the owner");
        _;
    }
// needs to end with _;
```

```solidity
//SPDX-License-Identifier: MIT

pragma solidity 0.8.3;

contract ModifierExample {

    mapping(address => uint) public tokenBalance;
    address owner;
    uint tokenPrice = 1 ether;

    constructor() {
        owner = msg.sender;
        tokenBalance[owner] = 100;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "You are not the owner");
        _;
    }

    function createNewToken() public onlyOwner {
        tokenBalance[owner]++;
    }

    function burnToken() public onlyOwner {
        tokenBalance[owner]--;
    }
}
```

We can take this one step further by abstracting it with Inheritance.

### Inheritance&#x20;

{% hint style="warning" %}
Inherited contracts are deployed as a single contract, not seperate ones.
{% endhint %}

We shall move the `owner` variable and `modifier` into a separate contract called Owned. Our earlier contract will now inherit as per `ModifierExample is Owned`

```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.3;

contract Owned {
    address owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "You are not the owner");
        _;
    }
}

contract ModifierExample is Owned {

    mapping(address => uint) public tokenBalance;
    uint tokenPrice = 1 ether;

    constructor() {
        tokenBalance[owner] = 100;
    }

    function createNewToken() public onlyOwner {
        tokenBalance[owner]++;
    }

    function burnToken() public onlyOwner {
        tokenBalance[owner]--;
    }
}

```

The derived contract is able to utilize the modifier `onlyOwner` defined in the parent as well as the `owner` variable which was set by the parent.&#x20;

When contract A inherits from contract B, only contract A is actually deployed to the blockchain. All code from contract B is copied into contract A. So there is only one smart contract address.&#x20;

{% hint style="info" %}
Solidity can do multiple inheritance (C3 linearization, Polymorphism)
{% endhint %}

### Import & Inherit

{% tabs %}
{% tab title="Owned.sol" %}
```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.3;

contract Owned {
    address owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "You are not the owner");
        _;
    }
}
```
{% endtab %}

{% tab title="ModifierExample.sol" %}
```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.3;

import "./Owned.sol";

contract ModifierExample is Owned {

    mapping(address => uint) public tokenBalance;
    uint tokenPrice = 1 ether;

    constructor() {
        tokenBalance[owner] = 100;
    }

    function createNewToken() public onlyOwner {
        tokenBalance[owner]++;
    }

    function burnToken() public onlyOwner {
        tokenBalance[owner]--;
    }
}
```
{% endtab %}
{% endtabs %}

[https://ethereum.stackexchange.com/questions/92265/solidity-proper-way-to-import-contracts](https://ethereum.stackexchange.com/questions/92265/solidity-proper-way-to-import-contracts)

* both functions and state variables will be available&#x20;
* With the `is` case, the contract **inherits** all the functionality of the base contract.&#x20;
* It means that when you instantiate the contract, a single object code (i.e., a single contract instance) containing all of that functionality is deployed on the network. Any call to a function of the base contract yields **a non-external** function call.

### **Import only (no inheritance)**

Factory pattern -> used to deploy instances of a contract via new keyword

{% hint style="info" %}
The new keyword in Solidity **deploys and creates a new contract instance**. It initializes the contract instance by deploying the contract, initializing the state variables, running its constructor, setting the nonce value to one, and, eventually, **returns the address of the instance to the caller.**
{% endhint %}

* importing a contract into another so that you can use its instances in the contract you are importing it in
* With the `has` case, the contract only holds a pointer to the other contract. They are essentially two different instances deployed on two different address on the network. Any call to a function of the other contract yields **an external** function call.

### Other import stuff

* [https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a](https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a)
* [https://betterprogramming.pub/learn-solidity-the-factory-pattern-75d11c3e7d29](https://betterprogramming.pub/learn-solidity-the-factory-pattern-75d11c3e7d29)
