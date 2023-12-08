# Core functions

### balanceOf()

![](<../../.gitbook/assets/image (226).png>)

* Allows you to check the token balance of an address. Returns token balance as uint.
* Public function that anyone can call.
* View function in that it does not modify the blockchain state - just reads from it.

## \_msgSender()

\_msgSender() is inherited from Context.sol, which is an abstract contract, defined as follows: function \_msgSender() internal view virtual returns (address) { return msg.sender; }

Initially, this seems trivial as we could directly call msg.sender. However, as the documentation explains, they should not be accessed in such a direct manner, since when dealing with meta-transactions the account sending and paying for execution may not be the actual sender (as far as an application is concerned).

For regular transactions it returns msg.sender and for meta transactions it can be used to return the end user (rather than the relayer).

## \_transfer()

We will need to understand this function well, before we can move to appreciating what either transfer() or transferFrom() do.&#x20;

In a nutshell, \_transfer() updates both the token balance states of `from` and `to` , as provided. It is called on by both transfer() and transferFrom() -- which are commonly used in other Smart Contracts.\


![](<../../.gitbook/assets/image (181).png>)

Both `_beforeTokenTransfer` and `_afterTokenTransfer` are hooks  — empty function shells where we can use to inject our own custom functionality; without modifying \_transfer() as it has been securely implemented.&#x20;

It is often useful to perform some function each time tokens change hands. To that end, we should not modify \_transfer as it is easy to write it insecurely. The solution therefore is to introduce hooks for custom functionality.

* First two require statements are sanity checks.
* The third ensures that you (address from), has the necessary balance to fulfil the transfer amount.
* Subsequently, the token sender’s balance is updated to reflect the post-transaction figure. This is wrapped inside **unchecked**{}, to save gas.
* The receiver’s token balance is incremented accordingly.
* Finally the Transfer event is emitted.

### A bit more on Unchecked:

The reason the "unchecked" keyword exists is to allow Solidity developers to write more efficient programs.

The default "checked" behaviour costs more gas when executing arithmetic operations, because under-the-hood those checks are implemented as a series of opcodes that, prior to performing the actual arithmetic, check for under/overflow and revert if it is detected.&#x20;

If you are working on Solidity v0.8.0 or greater, and you are able to determine there is no possible way for your arithmetic to under/overflow (in this case, our 3rd require statement), then you can surround the arithmetic in an "unchecked" block.

## \_mint()

![](<../../.gitbook/assets/image (175).png>)

* \_beforeTokenTransfer() and \_afterTokenTransfer are hooks - empty functions for customization.&#x20;
* **internal** modifier means that the function can only be called within the contract itself and any derived contracts.&#x20;
* In this form, anyone can call \_mint, which is not ideal. A typical implementation would be as follows, where the **onlyOwner modifier** is inherited from **Ownable.sol**

![onlyOwner can mint](<../../.gitbook/assets/image (179).png>)

## \_burn()

![](<../../.gitbook/assets/image (148).png>)

1. First is a sanity check, we obioculy cannot burn from a 0 address.&#x20;
2. \_beforeTokenTransfer is a hook, as is \_afterTokenTransfer&#x20;
3. Record current token balance of address&#x20;
4. Check that balance exceeds burn&#x20;
5. <mark style="color:orange;">**Unchecked**</mark> to save gas since require statement was qualified: subtract burn amt from address balance&#x20;
6. Update token’s total supply.&#x20;
7. Emit Transfer event.

Similar to \_mint, \_burn in this form would be callable by anyone, hence onlyOwner is useful.

## \_spendAllowance()

![](<../../.gitbook/assets/image (40).png>)

* Stores current allowance into `currentAllowance` variable
* If statement checks if `currentAllowance` is set to the maximum possible figure, else it ensures that `currentAllowance` is more than equals to the amount.
