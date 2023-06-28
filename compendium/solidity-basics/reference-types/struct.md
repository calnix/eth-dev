---
description: mappings + structs are a powerful combination
---

# struct

#### Picking up from the mapping example:

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.1;

contract MappingStructExmaple {

    struct Payment {
            uint amount;
            uint timestamp;
    }
    struct Balance {
        uint totalBalance;
        uint numPayments;
        mapping (uint => Payment) payments; //map numPayments to struct Payment - called payments
    }
    mapping(address => Balance) public balanceReceived;

    function getBalance() public view returns(uint){
        return address(this).balance;
    }

    function sendMoney() public payable {
        balanceReceived[msg.sender].totalBalance += msg.value;   // balance init @0. incremented from there.

        Payment memory payment = Payment(msg.value, block.timestamp);        //payment = Payment(amt, time)
        
        balanceReceived[msg.sender].payments[balanceReceived[msg.sender].numPayments] = payment;
        balanceReceived[msg.sender].numPayments++;
    }

    function partialWithdraw(uint _withdraw_amt, address payable _to) public {
        require(_withdraw_amt <= balanceReceived[msg.sender].totalBalance, "Balance exceeded");  //do you hav enuff to withdraw
            balanceReceived[msg.sender].totalBalance -= _withdraw_amt;       //update balances
            _to.transfer(_withdraw_amt);                                    // whr u want to send to?
    }

    function withdrawAllMoney(address payable _to) public {   //i can withdraw to addr of choice
        uint user_balance = balanceReceived[msg.sender].totalBalance;
        balanceReceived[msg.sender].totalBalance = 0;
        // checks-effect interaction pattern 
        _to.transfer(user_balance);
    }
}
```

### Create object:  **`Payment,`**

* Payment.amount&#x20;
* Payment.time

to reflect the value and datetime per transaction.&#x20;

### Create object: **`Balance`**,&#x20;

* Balance.totalBalance
* numPayments          _(initialized @ 0)_
* mapping(uint => Payment) payments

### Create : `mapping(address => Balance) public balanceReceived;`

Because mappings have no length, we can't do something like `balanceReceived.length` or `payments.length`. It's technically not possible. In order to store the length of the payments mapping, we have an additional helper variable `numPayments`.

So, if you want to the first payment for address 0x123... you could address it like this: `balanceReceived[0x123...].payments[0].amount = ...`. But that would mean we have static keys for the payments mapping inside the Balance struct. We actually store the keys in `numPayments`, that would mean, the current payment is in `balanceReceived[0x123...].numPayments`. If we put this together, we can do `balanceReceived[0x123...].payments[balanceReceived[0x123...].numPayments].amount = ...`.

{% hint style="warning" %}
numPayments **is not** number of payments made.&#x20;

no. of payments made = numPayments -1 (since we start frm 0) &#x20;

numPayments serves as an incrementing counter, to load the next "index" into the mapping to create a sequence starting with 0.&#x20;
{% endhint %}

#### balanceReceived

```solidity
    //mapping(address => uint) public balanceReceived;
    mapping(address => Balance) public balanceReceived;
```

In this new mapping, `balanceReceived[addr]` returns a struct:&#x20;

* `balanceReceived[addr].totalbalance`&#x20;
* `balanceReceived[addr].numPayments`

### sendmoney()

#### Updating total balance&#x20;

`balanceReceived[msg.sender].totalBalance += msg.value;`

`Balance.totalBalance += msg.value`

The first simplifies to the second line, where the Balance object is respective to the address fed in the first.

#### Updating individual payments as per ledger

Balance object, contains a mapping as a member, mapping (uint => Payment) payments; it maps an integer ("index") to an object Payment.

First we will create the individual transaction.

```solidity
// create the payment: insit + assign
// payment = Payment(amt, time)
Payment memory payment = Payment(msg.value, now);      //memory cos ref. type
```

Now we need to book the transaction into our "ledger", which is the mapping **`payments`**

```solidity
// booking .numPayment
// x= 0, on init
	balanceReceived[msg.sender].payments[x] = payment  //(Payment(msg.value, now))

the key x is:
	balanceReceived[msg.sender].numPayments (= 0, on init.)
	
nesting it:
	balanceReceived[msg.sender].payments[balanceReceived[msg.sender].numPayments] = payments
```

Loading the transaction is easy enough, from the first line. But how to create a running sequence?

We would need an index/counter -> `numPayments` -> key value of mapping **`payments`**.

* `uint numPayments` init as 0, so our counter starts at 0 for the first payment.

After booking a transaction, increment `numPayments` in preparation for the next transaction

* `balanceReceived[msg.sender].numPayments++;`

{% hint style="info" %}
Might wonder, why not use array? Gas costs; we rarely touch arrays.
{% endhint %}
