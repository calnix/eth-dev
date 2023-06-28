---
description: https://forum.openzeppelin.com/t/import-vs-import-and-inherit/4772/7
---

# Interface & Abstract Contracts

You can interact with other contracts by declaring an `Interface`.

* Interfaces can have only unimplemented functions.
* Interfaces can inherit from other interfaces only
* cannot declare a constructor, state variables or functions of its own.
* all declared functions must be **external**
  * interface serves as a passthrough to the contract you are interacting with
  * hence, external&#x20;

The interface serves as a filter; functions declared within it can be called as methods from the contract you wish to interact with.

You will need to pass the contract address into the interface object.

```solidity
// Example"

// import interface AggregatorV3Interface
import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";

contract FundMe {
    
    // declare state var
    AggregatorV3Interface public priceFeed;
    
    //constructor
    constructor(address _priceFeed) {
        priceFeed = AggregatorV3Interface(_priceFeed);
        owner = msg.sender;
    }
    
    function getPrice() public view returns (uint256) {
        (, int256 answer, , , ) = priceFeed.latestRoundData();
        return uint256(answer * 10000000000);
    }    
}
```

* declare priceFeed as an interface object&#x20;
* use constructor to pass the address into the interface,&#x20;
* store it into priceFeed to be used later for easy reference

<details>

<summary>Generic example</summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Counter {
    uint public count;

    function increment() external {
        count += 1;
    }
}

interface ICounter {
    function count() external view returns (uint);

    function increment() external;
}

contract MyContract {
    function incrementCounter(address _counter) external {
        ICounter(_counter).increment();
    }

    function getCount(address _counter) external view returns (uint) {
        return ICounter(_counter).count();
    }
}
```

</details>

### Summary

An interface is like a blueprint for a contract. It says what functions there are and what parameters they take but does not say what they actually do. If your contract inherits from an interface, the compiler will make sure that you do not forget to provide implementations for all its functions and that you do not change their signatures.

You can also use an interface to tell the compiler that a contract deployed at a particular address provides a particular set of functions. This way the compiler can know how to encode parameters for an external call without having access to the whole source code of the contract you’re calling.

## Abstract Contracts

Abstract contracts are a mix between a contract and an interface. They can have both unimplemented functions (like interfaces) and implemented ones (like contracts).

The use case is usually different though. You’d typically create an abstract contract if you wanted to create an “incomplete” contract that cannot be deployed on its own and where the one inheriting from it has to fill the gaps. That’s usually better than creating a contract with empty function stubs because the compiler can check if all the right functions have been provided by the inheriting contract.

Contracts are identified as abstract contracts if at least one of their functions lacks an implementation. -> in an interface all the functions are implemented.

* This is the only requirement for abstract class.
* They are used as base contracts from which other contracts can inherit from.

An abstract contract is one that cannot be deployed by itself. An abstract contract must be inherited by another contract.

### Difference between abstract contract and an interface

* An interface cannot have a constructor while an abstract contract can implement one.
* An interface cannot define state variables but an abstract contract can.
* An inheriting contract must implement all the functions defined in an interface while in an abstract contract the inheriting contract must implement at least one function of the abstract contract.
* An abstract contract can inherit from another contract or abstract contract while an interface cannot inherit from a contract or another interface.



## Implmenting Interfaces

[https://stackoverflow.com/questions/64733976/i-am-having-a-difficulty-of-understanding-interfaces-in-solidity-what-am-i-miss](https://stackoverflow.com/questions/64733976/i-am-having-a-difficulty-of-understanding-interfaces-in-solidity-what-am-i-miss)\
\
[https://medium.com/coinmonks/solidity-tutorial-all-about-interfaces-f547d2869499](https://medium.com/coinmonks/solidity-tutorial-all-about-interfaces-f547d2869499)

[https://www.geeksforgeeks.org/solidity-basics-of-interface/](https://www.geeksforgeeks.org/solidity-basics-of-interface/)
