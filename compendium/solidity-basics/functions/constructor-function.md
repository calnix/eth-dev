---
description: https://docs.soliditylang.org/en/v0.8.12/contracts.html
---

# Constructor Function

Constructors are a common concept in OOP. In many programming languages, when defining classes, you can also define a magic method that will run once, at the time a new instance of the object is created.

In the case of Solidity, the code defined inside the constructor will run only once, at the time the contract is created and deployed in the network.

> An important point to mention is the following:
>
> > The **bytecode** deployed on the network does not contain the constructor code, since the constructor code runs only once, on deployment.
>
> \
> [OpenZeppelin explains the idea more precisely in the second part of their article series “Deconstructing a Solidity Contract”](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-ii-creation-vs-runtime-6b9d60ecb44c/):
>
> > The constructor code is part of the creation code, not part of the runtime code.

#### Deployment

After the constructor has executed, the final code of the contract is stored on the blockchain. This code includes all public and external functions and all functions that are reachable from there through function calls. The deployed code does not include the constructor code or internal functions only called from the constructor.

### How to define a constructor in Solidity?

You define a constructor in Solidity with the keyword `constructor()` followed by parentheses. Note that you do not need to add `function` keyword, since it is a special function.

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
```

Before deployment, you have to provide the values for `feet` and `canSwim` &#x20;

![](<../../../.gitbook/assets/image (246).png>)

{% hint style="danger" %}
Prior to version 0.4.22 of Solidity, constructors were defined as functions with the same name as the contract (_like in JAVA?)_. This syntax was deprecated and is not allowed anymore since version 0.5.0.&#x20;
{% endhint %}

### Public vs Internal Constructors&#x20;

Prior to Solidity 0.7.0, contract’s constructors in Solidity could be defined with the following 2 visibilities: `public` (default) or `internal`.

From Solidity 0.7. onward:

* Visibility (`public` / `internal`) is not needed for constructors anymore: To prevent a contract from being created, it can be marked `abstract`.&#x20;
* This makes the visibility concept for constructors obsolete.

### Abstract contracts

An abstract contract enables the child contract that inherits from it to setup some default logic on deployment (specific to the abstract contract). This initial deployment logic can be dynamic, by declaring the constructor with parameters.&#x20;

* manner of inheritance ( i think: wen you want to inherit the logic, but define the init values for constructor)
* [https://medium.com/upstate-interactive/solidity-how-to-know-when-to-use-abstract-contracts-vs-interfaces-874cab860c56](https://medium.com/upstate-interactive/solidity-how-to-know-when-to-use-abstract-contracts-vs-interfaces-874cab860c56)

## Constructors’ parameters and inheritance <a href="#e6b3" id="e6b3"></a>

If a base contract has arguments, derived contracts need to specify all of them. This can be done in 2 ways:

#### **Directly in the inheritance list**

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

contract Lion is Animal(4, true) {
}
```

{% hint style="danger" %}
&#x20;If a derived contract does not specify the arguments to all of its base contracts’ constructors, it will be abstract.
{% endhint %}

#### **Through the derived constructor, like a **_**“modifier”**_ <a href="#bee5" id="bee5"></a>

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
    
    constructor(string memory _name)
        Animal(_name, 4, true)
    {
        // ...
    }
}
```

This way can be used if the constructor arguments of the base contract depend on those of the derived contract.

## Payable Constructor <a href="#882b" id="882b"></a>

Constructors can accept Ethers. In this case, they should be mentioned with the keyword “payable”. In the Remix IDE, the “deploy” button will change color (shown in red) if the constructor accept ether.

If you try to add ether when deploying a contract defined with a non payable constructor, it will throw you an exception and revert.

The difference between a payable and non payable constructor is also reflected in the creation code.

If the constructor is non-payable, the creation code contains 8 low-levels instructions (in EVM assembly), that check if some ethers has been sent to the contract once it has been deployed. If it is the case, the check fails and the contract reverts.

You can see the example here: [https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-ii-creation-vs-runtime-6b9d60ecb44c/](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-ii-creation-vs-runtime-6b9d60ecb44c/)

If the constructor is defined as **payable** , this 8 EVM instructions are removed and not present.\
