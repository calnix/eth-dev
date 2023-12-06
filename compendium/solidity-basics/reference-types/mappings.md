---
description: https://medium.com/coinmonks/solidity-tutorial-all-about-mappings-29a12269ee14
---

# mappings

* key-value pair
* a mapping is a HashMap (JS) -> no index -> similar to dictionary in python
* no indexing/order -> myMapping\[1] will not return the "first book"
* mappings don't have a length &#x20;

Mappings are accessed like arrays, but there are no index-out-of-bounds-exceptions. All possible key/value pairs are already initialized.&#x20;

## Initialization & Assignment

On Initialization, all possible key/value pairs are already initialized. You can simply access any key and get back "false" or 0 - whichever the default value is for the value type.

This is because the hash that points to the location in storage has not been generated & we have not written a value thus far.

#### Assignment of values must be done inside a function

* cannot simply do `myMapping[1] = "test"`  (like python)

### Example

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract SimpleMappingExample {

    mapping(uint => bool) public myMapping;
    mapping(address => bool) public myAddressMapping;

    function setValue(uint _index) public {
        myMapping[_index] = true;
    }

    function setMyAddressToTrue() public {
        myAddressMapping[msg.sender] = true;
    }
}
```

![](<../../../.gitbook/assets/image (110).png>)         ![](<../../../.gitbook/assets/image (7) (1).png>)

1. On initialization, `myMapping[1]` will return the default bool value of false.&#x20;
2. Then we call `setValue` setting -> `myMapping[1]` = true
3. Subsequently, when we call `myMapping[1]`, we are returned true as set earlier.

#### Whitelisting&#x20;

If an address is allowed to do a certain action in our Smart Contract then we can white-list it. To achieve whitelisting, we can map addresses to boolean values.&#x20;

On initialization, any address passed into the mapping would return false: `myAddressMapping`

We can whitelist using a function variation on the example: `setMyAddressToTrue()`

## Nested Mapping

```solidity
//not public no getter function
mapping (uint => mapping(uint => bool)) uintUintBoolMapping; 

// getter for nested mapping
function getNestedMap(uint _key1, uint _key2) public view returns(bool){
    return uintUintBoolMapping[_key1][_key2];

//setter
function setUintUintBoolMapping(uint _index1, uint _index2, bool _value) public {
    uintUintBoolMapping[_index1][_index2] = _value;
    }
```

For the nested mapping above, the outer mapping maps a host of integers to a set of mappings.&#x20;

* outer_mapping\[1] -> inner_mapping\_1
* outermapping\[2] -> innermapping\_2\[?] ->  value?

Each key leads to a different inner mapping. As such, you would need a 2nd parameter (\_key2) for the inner mapping.&#x20;

![](<../../../.gitbook/assets/image (296).png>)

## Key and value types allowed <a href="#3d0a" id="3d0a"></a>

Not every data type can be used as a key ->  **`struct`** and **`mapping`** cannot be used as keys.

Solidity does not limit the data type for values. It can be anything, including `struct` and `mapping.`

![](<../../../.gitbook/assets/image (130).png>)

### Example: tracking balances and withdrawals

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.1;

contract MappingStructExmaple {

    mapping(address => uint) public balanceReceived;

    function getBalance() public view returns(uint){
        return address(this).balance;
    }

    function sendMoney() public payable {
        balanceReceived[msg.sender] += msg.value;   // balance init @0. incremented from there.
    }

    function partialWithdraw(uint _withdraw_amt, address payable _to) public {
        require(_withdraw_amt <= balanceReceived[msg.sender], "Balances exceeded");  //do you hav enuff to withdraw
            balanceReceived[msg.sender] -= _withdraw_amt;       //update balances
            _to.transfer(_withdraw_amt);                        // whr u want to send to?
    }

    function withdrawAllMoney(address payable _to) public {   //i can withdraw to addr of choice
        uint user_balance = balanceReceived[msg.sender];
        balanceReceived[msg.sender] = 0;
        // checks-effect interaction pattern 
        _to.transfer(user_balance);
    }
}
```

Our accounting ledger is the mapping balanceReceived.&#x20;

{% hint style="info" %}
```
:: To avoid re-entracy bug:: checks-effect-interaction pattern
1) check if you can do something - condition 
2) then ensure the internal state is updated
---- internal first. external last. protecc us.
3) finally, external interactions.

https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html
```
{% endhint %}

## Iterable mappings can be implemented using libraries

