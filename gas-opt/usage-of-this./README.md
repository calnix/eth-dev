# usage of this.

### Example

```solidity
function unwrap(uint256 amount) external {
    require(this.balanceOf(msg.sender) >= amount, "Not Enough WRings to unwrap");
    
    burn(msg.sender, amount);
    iRings.transferFrom(address(this), msg.sender, amount);
    emit Unwrapped(msg.sender, amount);
}
```

* Using <mark style="color:yellow;">`this`</mark> to call functions **in the same contract** makes the call an external call.&#x20;
* That means an extra 2100 gas, and rewriting the `msg.*` variables.&#x20;
* better off not using `this.*` until you find the very few use cases.

#### Note if you drop <mark style="color:yellow;">`this`</mark> :

* `require(`**`balanceOf`**`(msg.sender) >= amount, "Not Enough WRings to unwrap"`
* `it will not work`

That is because **`balanceOf`** is **external**, to stop developers using suboptimal (in terms of gas) patterns.&#x20;

* since the contract **inherits** ERC20, **`balanceOf`** is native to itself; contract cannot call its own external function,

![ERC20: balanceOf()](<../../.gitbook/assets/image (343).png>)

### Solution:

You should access the `_balanceOf` mapping instead. It is `internal`

![ERC20.sol](<../../.gitbook/assets/image (340).png>)

