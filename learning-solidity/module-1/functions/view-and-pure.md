# View and Pure

### View

The view keyword is a modifier used in Solidity to indicate that a function does not modify the state of the contract. View functions are able to access values from storage, but cannot modify them.&#x20;

* It allows the function to call other functions which do not modify the state, while still disallowing other functions which do modify the state.
* The view keyword can also be used to make contracts more **secure**, as it ensures that data cannot be modified accidentally.

```solidity
contract MyContract {    
    uint256 public myValue; 
    
    // view function
    function getMyValue() public view returns (uint256) {        
        return myValue;    
    }
}
```

* Since no state changes are made, the means that view functions can be called without incurring any gas costs.

#### Conditions for a statement to be considered as “modifying the state”:

1. State variables being written to.
2. Events being emitted.
3. Other contracts being created.
4. `selfdestruct` being used.
5. ETH being sent via calls.
6. Calling functions that are not marked view or pure.
7. Low-level calls being used.
8. Inline assembly containing certain opcodes being used

{% hint style="info" %}
Events don't change the state of the contract where they are emitted but they change the state of the whole blockchain.

\
In simple terms, a log (emitting an event) will make a change in the **Transaction Receipts Root** that is recorded in the header of a block and a adding new block is considered a change in the state of the blockchain.
{% endhint %}

### Pure

Pure function is one that does not interact with the blockchain in any way, including reading state of variables. It is self-encapsulated.&#x20;

**A pure function does not:**

* Read or write to the blockchain
* Create or send a transaction
* Access any state variables
* Call any other functions

```solidity
function convertWeiToEther(uint _amountInWei){
	return _amountInWei / 1 ether;
}
```

{% hint style="warning" %}
A pure function can ONLY call another pure function.
{% endhint %}
