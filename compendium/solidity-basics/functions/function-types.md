# Function types

```solidity
function (<parameter types>) {public|private|internal|external} 
[pure|view|payable] [(modifiers)] [returns (<return types>)]
```

#### A function declarations only impose scope and do nothing for security

![](<../../../.gitbook/assets/image (52).png>)

#### Simply put:

**public ->** all can access

**external ->** only externally calls; cannot be accessed internally

**internal ->** only this contract and contracts deriving from it can access

**private ->** can only be accessed from within this contract

### View

View functions do not modify the state, thus only being used for viewing the state. Therefore only can return data:

* List of conditions for a statement to be considered as “modifying the state”:
  1. State variables being written to.
  2. Events being emitted.
  3. Other contracts being created.
  4. `selfdestruct` being used.
  5. ETH being sent via calls.
  6. Calling functions that are not marked view or pure.
  7. Low-level calls being used.
  8. Inline assembly containing certain opcodes being used

A View function can call on other View or Pure functions. \
Cannot call a writing f() -> else it would no longer be a read-only interaction.

### Pure

pure function is one that does not interact with the blockchain in any way, including reading state of variables. It is self-encapsulated.&#x20;

```solidity
function convertWeiToEther(uint _amountInWei){
	return _amountInWei / 1 ether;
}
```

A pure f() can ONLY call another pure f() -> anything else it would violate the rule and interact w/ SC.

{% hint style="info" %}
a writing f() can call any.
{% endhint %}
