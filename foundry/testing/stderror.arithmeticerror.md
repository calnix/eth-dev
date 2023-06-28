# stdError.arithmeticError

### Original Implementation

{% code title="BasicVault.sol" %}
```solidity
    function withdraw(uint amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance!");
        
        emit Withdrawal(msg.sender, amount);
        balances[msg.sender] -= amount;
        
        (bool success) = wmdToken.transfer(address(this), amount);
        require(success, "Token transfer frm User to Vendor failed!");
    }
```
{% endcode %}

{% code title="BasicVaultTest.t.sol" %}
```solidity
    function testUserCannotWithdraw(uint amount) public {
        console2.log("User cannot withdraw with no balance");
        vm.assume(amount > 0);
        vm.prank(user);
        vm.expectRevert("Insufficient balance!");
        vault.withdraw(amount);
    }
```
{% endcode %}

#### Alberto said:

> testUserCannotWithdraw and testUserCannotWithdrawExcessDeposit should revert because of underflow in [balances\[msg.sender\]](https://github.com/calnix/Basic-Vault/blob/06718a6786105571f362101e990f844effe6b42c/src/BasicVault.sol#L49), not because of ERC20 properties.

#### Richie elaborated:

> Alberto said he wanted to see this revert because of the underflow, not because of the erc20 balance. so u can:
>
> 1. make sure the user has an balance of the token
> 2. change this to vm.assume`(amount > 0 && amount < wmd.balanceOf(user))`
> 3. instead of expectRevert -- use forge-std standard errors [https://book.getfoundry.sh/reference/forge-std/std-errors.html](https://book.getfoundry.sh/reference/forge-std/std-errors.html)

### Correct Implementation



{% code title="RevisedCode: BasicVault.sol" %}
```solidity
    function withdraw(uint amount) external {      
        balances[msg.sender] -= amount;
        
        bool success = wmdToken.transfer(msg.sender, amount);
        require(success, "Withdrawal failed!");
        emit Withdrawal(msg.sender, amount);
    }
```
{% endcode %}

{% code title="RevisedCode: BasicVaultTest.t.sol" %}
```solidity
function testFuzzUserCannotWithdraw(uint amount) public {
    console2.log("User cannot withdraw with no balance");
    vm.startPrank(user);
    wmd.mint(user, amount);
    vm.assume(amount > 0 && amount < wmd.balanceOf(user));
    
    vm.expectRevert(stdError.arithmeticError);
    vault.withdraw(amount);
    vm.stopPrank();
}
```
{% endcode %}

### The takeaway here is that:

1. the user has some wmd tokens.
2. he will attempt to withdraw some tokens within the range (0, amount), from the vault
3. since he does not have any deposits to begin with, we run into an underflow error, resulting from:\
   `balances[msg.sender] -= amount;`

as this is going to work out to subtracting a positive figure (amount) from 0 -> hence underflow.
