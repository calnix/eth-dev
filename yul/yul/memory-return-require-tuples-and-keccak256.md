---
description: >-
  https://github.com/RareSkills/Udemy-Yul-Code/blob/main/Video-10-Require-Return-Tuple-Keccak256.sol
---

# Memory: Return, Require, Tuples and Keccak256

### Return

```solidity
contract UsingMemory {

    function return2and4() external pure returns (uint256, uint256) {
        assembly {
            mstore(0x00, 2)
            mstore(0x20, 4)
            return(0x00, 0x40)        //returns 2 and 4
        }
    }
}
```

* return operates as a function within Yul, not as a keyword that we are used to
* return in yul returns an area in memory, specified by arguments
* return(start\_mem\_addr, end\_mem\_addr)

### Revert

* like return, revert can return a specified area in memory&#x20;
* revert allows for execution to be reverted, and return some data so that the calling function can respond and do something with it
* if you just want execution to stop: `revert(0, 0)`
