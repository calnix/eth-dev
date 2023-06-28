# Calling a view function

A `view` function does not use `gas` when called, but if we decide to call a view function inside of another function which is a transaction, it then uses `gas`.

{% hint style="info" %}
Even if you make a function a `view` function, you still have to pay full gas costs if it's being called inside a transaction. Such `view` functions are only free if called locally, not inside a transaction.
{% endhint %}

```solidity
//SPDX-License-Identifier:MIT

pragma solidity ^0.8.3;
contract GasSaver {
    uint256[] private numbers = [2,3,5,67,34];

    function getNumberAt( uint256 _index ) public view returns (uint256){
        return numbers[_index];
    }
    //Gas used when getNumber was called: 44778

    //Gas used without calling getNumber() 44450
    function sumAndMultiply() public {
       uint256[] memory _numbers = numbers;
       uint256 arrlength = _numbers.length;
       for(uint256 i=0; i < arrlength; ++i){
          numbers[i] = _numbers[i] * i;
       }
       //getNumberAt(2);
    }
}
```

* calling `getNumberAt` uses no gas

\
