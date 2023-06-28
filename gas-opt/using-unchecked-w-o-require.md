# using unchecked w/o require

## Integer overflow/underflow

Prior to Solidity `0.8.0` version, interger overflow and underflow checks were performed by using the `SafeMath` library. From Solidity `0.8.0` upward, the compiler does that check for us.\


This extra check cost `gas`. If we know that the mathematical operations we will perform inside the contract will not overflow or underflow, we can tell the compiler not to check for overflow or underflow in the operation.\


This is the purpose of unchecked.

### Typical Implementation&#x20;

```solidity
    function deposit(uint256 amount) public {
        balances[msg.sender] += amount;
        bool successful = token.transferFrom(msg.sender, address(this), amount);
        require(successful, "Deposit failed!"); 
        emit Deposited(msg.sender, amount);
    }
```

### Could you use `unchecked` here?&#x20;

* Yes, it would be safe -> as long as the <mark style="color:yellow;">deposited token is known to be not malicious.</mark>
* The deposited balance of any account can't ever be more than the total supply of the deposited token. No one can own more than `type(uint256).max` of the underlying token.
* I'm being a bit pedantic in here, but I wanted to point out that sometimes conditions are enforced by construction, instead of being enforced by `require` statements.

### new implementation

```solidity
    function deposit(uint256 amount) public {
        unchecked { balances[msg.sender] += amount; }
        bool successful = token.transferFrom(msg.sender, address(this), amount);
        require(successful, "Deposit failed!"); 
        emit Deposited(msg.sender, amount);
    }
```

* unchecked saves gas.
