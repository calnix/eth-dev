# Syntactic Sugar

### Incrementation

There are four ways to increment and decrement a variable in Solidity.

```solidity
//SPDX-License-Identifier:MIT
pragma solidity ^0.8.3;

contract test1{
   uint256 public number;

    // transaction cost: 43816
    function increment() public returns (uint256){
        number += 1;
        return number;
   }
}

contract test2{
   uint256 public number;

    // transaction cost: 43803
    function increment() public returns (uint256){
        number = number + 1;
        return number;
   }
}

contract test3{
   uint256 public number;

    // transaction cost: 43653 
    function increment() public returns (uint256){
        return number++;
   }
}

contract test4{
   uint256 public number;

   // transaction cost: 43647 
    function increment() public returns (uint256){
        return ++number;
   }
}
```

* Above list four contracts illustrating how we could increment a number variable by a value of 1.&#x20;
* Contract `test4` is the most gas effective out of the lot when ran.&#x20;

### Decrementing

```solidity
//SPDX-License-Identifier:MIT
pragma solidity ^0.8.3;

contract test1{
   uint256 public number = 1;

    // transaction cost: 21894 
    function decrement() public returns (uint256){
        number -= 1;
        return number;
   }
}

contract test2{
   uint256 public number = 1;

    // transaction cost: 21881 
    function decrement() public returns (uint256){
        number = number - 1;
        return number;
   }
}

contract test3{
   uint256 public number = 1;

    // transaction cost: 21731  
    function decrement() public returns (uint256){
        return number--;
   }
}

contract test4{
   uint256 public number = 1;

   // transaction cost: 21725  
    function decrement() public returns (uint256){
        return --number;
   }
}
```

* Once again `test4` approach requires the least gas.
