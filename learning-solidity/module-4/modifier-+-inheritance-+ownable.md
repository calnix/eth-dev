# Modifier + Inheritance +Ownable

* Modifiers are code that can be run before and / or after a function call.
* Modifiers can be used to restrict access to certain functions or to enforce business logic. Here are some examples of how modifiers can be used.

**Example**

{% tabs %}
{% tab title="pre-check" %}
```solidity
modifier onlyOwner() {
   require(msg.sender == owner, "Not owner");
   // Underscore is a special character only used inside
   // a function modifier and it tells Solidity to
   // execute the rest of the code.
   _;
}
```
{% endtab %}

{% tab title="post-check" %}
```solidity
// This modifier prevents a function from being called while
// it is still executing
modifier noReentrancy() {
    require(!locked, "No reentrancy");

    locked = true;
    _;                // function executes
    locked = false;
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
If you are repeating a conditional check in multiple functions, it can be wrapped within a modifier for easy reuse.
{% endhint %}

{% tabs %}
{% tab title="Code" %}
{% code lineNumbers="true" %}
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
        require(msg.sender == owner, "You are not the owner");
        tokenBalance[owner]--;
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Code w/ modifier" %}
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
{% endtab %}
{% endtabs %}

* Both functions `burnToken()` and `createNewToken()` have the same require statement. We can centralize this by using a modifier instead.
* We can take this one step further by abstracting it with Inheritance.

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

