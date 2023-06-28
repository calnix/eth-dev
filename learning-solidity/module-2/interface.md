# Interface

An interface in Solidity is a collection of function definitions that a contract can implement. An interface is a way for a contract to communicate with other contracts or with external applications.

**Example**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Counter {
    uint public count;

    function increment() external {
        count += 1;
    }
}

// interface for Counter.sol
interface ICounter {
    function count() external view returns (uint);

    function increment() external;
}
```

**Properties**

* Interfaces cannot have any functions implemented
* Interfaces contain only **external** function declarations and state variables.
* Interfaces can only inherit from other interfaces.
* Interfaces do not necessarily have to list all of the external functions of a contract.

#### Summary

An interface is like a blueprint for a contract. It says what functions there are and what parameters they take but does not say what they actually do. If your contract inherits from an interface, the compiler will make sure that you do not forget to provide implementations for all its functions and that you do not change their signatures.

You can also use an interface to tell the compiler that a contract deployed at a particular address provides a particular set of functions. This way the compiler can know how to encode parameters for an external call without having access to the whole source code of the contract you’re calling.
