# <  vs  <=

* EVM has less than (LT), and greater than (GT) opcodes,
* But no `>=` or `<=` opcodes
* So less/greater than will be synthesized with the available opcodes, instead of a single opcode
* This increases gas costs

```solidity
    // less gas intensive
    function lessThan(uint256 x) external pure returns(bool){
        return x < 3;
    }

    // more gas intensive
    function lessThanEq(uint256 x) external pure returns(bool){
        return x <= 2;  //!(2 < x)
    }
```

* EVM process `x <=2` as `!(2 < x)`
