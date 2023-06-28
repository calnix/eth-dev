# msg.value & payable functions

## Payable functions

Say you want to make a call to **another contract's payable function** and transfer ETH:

```css
targetAddress.someFunction{ value: amount }(arg1, arg2, arg3)
```

* calls someFunction on targetAddress contract
* sends amount of ETH
* this works if someFunction is a payable function

### Calling payable functions

{% tabs %}
{% tab title="WETH.sol" %}
```solidity
function deposit() public payable {
    balanceOf[msg.sender] += msg.value;
    emit Deposit(msg.sender, msg.value);
}
```
{% endtab %}

{% tab title="Vault.sol" %}
```solidity
weth.deposit{value: 1 ether}();
```
{% endtab %}
{% endtabs %}

* From Vault, we call the payable function `deposit`, and pass msg.value within the `value` field
* [https://ethereum.stackexchange.com/questions/60279/how-to-set-msg-value-in-solidity-function-call](https://ethereum.stackexchange.com/questions/60279/how-to-set-msg-value-in-solidity-function-call)

