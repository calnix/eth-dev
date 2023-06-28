# Conditional(ternary) operators

```solidity
function _isCollateralized(uint debtAmount, uint collateralAmount) internal view returns(bool collateralized) {
       return debtAmount == 0 || debtAmount <= _collateralToDebt(collateralAmount);
    }
```

1. Conditions are evaluated sequentially.     &#x20;
2. First check if debtAmount == 0, if so, return TRUE.  If debtAmount **does not** equal to 0 (FALSE), proceed to evaluate 2nd conditional.
3. Second conditional checks that debtAmount is less than the max possible debt a user can take on given their deposits.
4. If second conditional evaluates to be true, TRUE is returned.
5. Else, FALSE is returned.

Essentially, the common short-circuiting rules would apply with the use of the || operator - such that if the first conditional is successfully evaluated to be true, subsequent conditionals will not be evaluated. (edited)

{% hint style="info" %}
[https://ethereum.stackexchange.com/questions/72641/logical-or-operator-is-not-working-in-if-condition-solidity-0-5-0](https://ethereum.stackexchange.com/questions/72641/logical-or-operator-is-not-working-in-if-condition-solidity-0-5-0)\

{% endhint %}



### if condition true ? then A : else B

```solidity
contract SolidityTest{
 
     // Defining function to demonstrate
     // conditional operator
     function sub(uint a, uint b) public view returns(uint){
      uint result = (a > b ? a-b : b-a);
      return result;
 }
```

Evaluates the expression first then checks the condition for return values corresponding to true or false.

#### Example:

```solidity
    function getExchangeRate() internal view returns(uint256) {
        uint256 sharesMinted;
        return _totalSupply == 0 ? sharesMinted = 1e18 : sharesMinted = (_totalSupply * 1e18) / underlying.balanceOf(address(this));
    //    if(_totalSupply == 0) {
    //        return sharesMinted = 1;
    //    }
    //    return sharesMinted = (_totalSupply) / underlying.balanceOf(address(this));
    }
```





#### Further reading

* [https://www.geeksforgeeks.org/solidity-operators/](https://www.geeksforgeeks.org/solidity-operators/)
* [https://medium.com/coinmonks/solidity-fundamentals-1fb0e6b3b607#:\~:text=XOR%20%2C%20NOT%20etc.-,Conditional%20Operators,if%20the%20condition%20is%20falsy.](https://medium.com/coinmonks/solidity-fundamentals-1fb0e6b3b607)
