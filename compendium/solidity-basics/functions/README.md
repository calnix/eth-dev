# Functions

**Function Structure**

![Example](<../../../.gitbook/assets/image (154).png>)

* `public` -> visibility declaration&#x20;
* `view` -> Indicates function behavior
* `returns` -> type of variable being returned

```solidity
 // updates message variable. cannot return.
    function setMessage(string newMessage) public {
        message = newMessage;
    }
    
 // synthax will clear. looks correct. but wont work.
    function setMessage(string newMessage) public returns (string){
        message = newMessage;
        return message;
    }    
```

{% hint style="danger" %}
You cannot create a function with the same name as a storage variable. Will conflict with getter function.
{% endhint %}

{% hint style="info" %}
**There are 2 types of functions that you can create in Solidity:**

1. Functions that create transaction on block chain
2. Functions that do not create transaction on block chain (view and pure)

_For more details see: Running Functions page_
{% endhint %}
