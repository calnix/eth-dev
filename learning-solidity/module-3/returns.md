# Returns

## Returns

* Functions can have an optional return statement; not restricted to pure & view only.&#x20;
* If function is not (pure,view,constant) -> returned-value are restricted to on-chain usage.
* If function is (pure,view,constant) -> returned-values can be off-chain.

### Functions without `pure`, `view`, `constant` modifiers

Returned values **can only be fetched by on-chain calls** (contract functions), but not by off-chain calls (Externally Owned Address/wallets).

When you call such function from off-chain you are **returned the transaction hash.**

* This is because it is unknown when the transaction will be mined and added to the blockchain.
* Only when it is successfully mined and computed, do we reach the actual value of the returns statement in the function.&#x20;
* This value from the returns statement can be used **only inside the blockchain** (contract functions).
* If you want this value to be accessible off-chain, you need to generate an `event` (inside the Solidity function) to emit it to listeners.

{% hint style="warning" %}
_No way for a transaction from an EOA to return a value that can be read, even by examining the blockchain._
{% endhint %}

### Functions that are `pure`, `view`, `constant`&#x20;

* These functions do not alter states or interact with the blockchain. As such, there is no transaction hash involved, as nothing is submitted for mining.&#x20;
* These "read-only" functions can return values for off-chain usage.&#x20;

### Example

If you want to `return true`, as below, the user interacting with it will not receive it as output. To achieve this, use events.

{% tabs %}
{% tab title="Return" %}
```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.3;

contract EventExampel {
mapping(address => uint) public tokenBalance;

constructor(){
    tokenBalance[msg.sender] = 100;
    }

function sendToken(address _to, uint _amount) public returns(bool){
    require(tokenBalance[msg.sender] >= _amount, "Not enough tokens");
    tokenBalance[msg.sender] -= _amount;
    tokenBalance[_to] += _amount;

    return true;
    }
}
```
{% endtab %}

{% tab title="Events" %}
```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.3;
contract EventExampel {

mapping(address => uint) public tokenBalance;
event TokenSent(address _from, address _to, uint _amount);

constructor(){
    tokenBalance[msg.sender] = 100;
}

function sendToken(address _to, uint _amount) public returns(bool){
    require(tokenBalance[msg.sender] >= _amount, "Not enough tokens");
    tokenBalance[msg.sender] -= _amount;
    tokenBalance[_to] += _amount;

    emit TokenSent(msg.sender, _to, _amount);
    return true;
    }
}
```
{% endtab %}
{% endtabs %}

* Create event `TokenSent`, taking in parameters that we want to emit to the outside world.&#x20;
* Then we emit the event before we are returning from the function.&#x20;
