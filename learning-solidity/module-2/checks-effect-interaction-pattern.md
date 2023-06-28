# Checks-effect-interaction pattern

<mark style="color:red;">**Why does it matter?**</mark>

When calling an external address, for example when transferring tokens to another account, the calling contract is also transferring the control flow to the external entity.&#x20;

Assuming this external entity is a smart contract as well, the <mark style="background-color:yellow;">**external entity is now in charge of the control flow**</mark> and can execute any inherent code within it.

**This can leave your contract open to re-entrancy attacks.**&#x20;

#### Negative Example

```solidity
function withdrawToken(uint256 amount) public {
    require(userBalance[msg.sender] >= amount, "requested amount exceeds user balance");
    
    // external call
    bool success = wmdToken.transfer(msg.sender, amount);
    require(success, "Withdrawal failed!");

    // internal state update
    userBalance[msg.sender] -= amount;
}
```

* `wmdToken.transfer` could be maliciously coded to call `withdrawToken` again, and the contract would transfer more tokens out without modifying the user balance.&#x20;
* The attacker would only have to be careful of stopping the loop before running out of gas, and it would drain your contract of iToken.

### Solution

The high-level idea is as follows:

1. check & update the internal states (balances)
2. keep external interactions (function calls) to the last step

#### Positive Example

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
