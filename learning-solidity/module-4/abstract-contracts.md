# Abstract contracts

* An abstract contract is a special contract type in Solidity that provides a base contract from which other contracts can be derived.&#x20;
* The purpose of an abstract contract is to provide various common and reusable pieces of code that can be used to create new contracts. This makes it easier for developers to create contracts that are more efficient and secure.
* For example OpenZepplin has a library of abstract contracts detailing out commonly used functionality, like [Ownable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol). These are typically imported and inherited by projects looking to implement such features.

**Example**

```solidity
pragma solidity ^0.8.0;

abstract contract Feline {
    function utterance() public virtual returns (bytes32);
}

contract Cat is Feline {
    function utterance() public override returns (bytes32) { return "meow"; }
}
```

* Base functions can be overridden by inheriting contracts to change their behavior if they are marked as `virtual`.
* The overriding function must then use the `override` keyword in the function header.&#x20;

{% hint style="info" %}
An abstract contract is one that cannot be deployed by itself. An abstract contract must be inherited by another contract.
{% endhint %}
