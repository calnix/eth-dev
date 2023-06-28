# Inheritance

In Solidity, inheritance works by means of a keyword, `is`, which is used to indicate that one contract inherits from another.

**Example**

```solidity
contract A {    
    function foo() public {        // do something    }
} 

contract B is A {    
    function bar() public {        // do something else    }
}
```

* Contract B inherits from the contract A.&#x20;
* This means that B will have access to the `foo()` function defined in A. In addition, it will also have its own `bar()` function.

### Import + Inherit

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

* ModifierExample contract **inherits** all the functionality of the base contract.
* It means that when you instantiate the contract, a single object code (i.e., a single contract instance) containing all of that functionality is deployed on the network. Any call to a function of the base contract yields **a non-external** function call.
* On deployment, there is only a single address.
