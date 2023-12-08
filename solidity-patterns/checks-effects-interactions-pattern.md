# checks-effects-interactions pattern

## Objective: Negate Reentrancy attacks

![](<../.gitbook/assets/image (135).png>)

### Negative Example

```solidity
function withdrawToken(uint256 amount) public {
    require(userBalance[msg.sender] >= amount, "requested amount exceeds user balance");
    require(iToken.transferFrom(address(this), msg.sender, amount), "failed on withdrawal");
    userBalance[msg.sender] -= amount;
```

* `iToken.transferFrom` could be maliciously coded to call `withdrawToken` again,&#x20;
* and the contract would transfer iToken out in a loop without modifying the user balance.&#x20;
* The attacker would only have to be careful of stopping the loop before running out of gas, and it would drain your contract of iToken.

### Positive Example

```solidity
    function withdraw(uint amount) external {      
        balances[msg.sender] -= amount;
        
        bool success = wmdToken.transfer(msg.sender, amount);
        require(success, "Withdrawal failed!");
        emit Withdrawal(msg.sender, amount);
    }
```

### Further Readings

* [https://www.securing.pl/en/reentrancy-attack-in-smart-contracts-is-it-still-a-problem/](https://www.securing.pl/en/reentrancy-attack-in-smart-contracts-is-it-still-a-problem/)
