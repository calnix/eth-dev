---
description: Use constant and immutable variables for variable that don't change
---

# constant and immutable

* Using the `constant` and the `immutable` keywords for variables that do not change helps to save on `gas` used.&#x20;
* The reason been that `constant` and `immutable` variables **do not occupy a storage slot** when compiled. They are saved inside the contract byte code.&#x20;

```solidity
//SPDX-License-Identifier:MITs
pragma solidity ^0.8.3;

interface IERC20 {    
function transfer(address to, uint256 value) external returns (bool);    
function approve(address spender, uint256 value) external returns (bool);    
function transferFrom(address from, address to, uint256 value) external returns (bool);   
//rest of the interface code
}

//Gas used: 187302
contract Expensive {

   IERC20 public token;
   uint256 public timeStamp = 566777;

   constructor(address token_address) {
       token = IERC20();
   }
}

//Gas used: 142890
contract GasSaver {
   IERC20 public immutable i_token;
   uint256 public constant TIMESTAMP = 566777;

   constructor(address token_address) {
       i_token = IERC20(token_address);
   }
}
```

* [https://blog.soliditylang.org/2020/05/13/immutable-keyword/](https://blog.soliditylang.org/2020/05/13/immutable-keyword/)

## Summary

```solidity
    IERC20 public immutable token; 
```

* For constant variables, the value has to be fixed at compile-time,&#x20;
* For immutable, it can still be assigned at construction time.
* `immutable` is one of the easiest gas optimizations that can be made, and at the same time very effective.
