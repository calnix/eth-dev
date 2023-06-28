# Data Location and Assignment Behaviors

Every reference type contains information on where it is **stored**. There are three possible options: `memory`, `storage`, and `calldata`. The set location is important for semantics of assignments, not only for the persistence of data:

`storage`: The location type where the state variables are stored on blockchain which means types that has `storage` location are persistent.

`memory`: Variables are in `memory` and they exists during the function call which means variables that got this location are temporary and after function execution finished, they won’t exist.

`calldata`: Non-modifiable, non-persistent data location **where function arguments are stored,** behaves mostly like `memory` data location and only available for `external` functions.&#x20;

* Assignments between `storage` and `memory` (or from `calldata`) always **generate an independent copy.**
* Assignments from `memory` to `memory` only create **references**. This means that changes to one memory variable are also visible in all other memory variables that refer to the same data.
* Assignments from `storage` to a **local storage** variable only assigns a reference.
* All other assignments to `storage` always creates independent copies. Examples for this case are assignments to state variables or to members of local variables of storage struct type, even if the local variable itself is just a reference.

#### Example: memory to memory

```solidity
contract localVar {

    function play() public pure returns(bytes memory) {
        bytes memory data;
        bytes memory meal = hex"f00df00d";

        data = meal;
        data[3] = 0xed;

        return meal;        // 0xf00df0ed
    }
}
```

* modifying `data`, also modified `meal`

## TLDR

* use `calldata` when you only need read-only data, avoiding the cost of allocating memory or storage.
* use `memory` if you want your argument to be mutable.
* use `storage` if your argument will already exist in storage, to prevent copying something into storage over into memory unnecessarily.
* [https://ethereum.stackexchange.com/questions/107028/in-what-cases-would-i-set-a-parameter-to-use-storage-instead-of-memory/123692#123692](https://ethereum.stackexchange.com/questions/107028/in-what-cases-would-i-set-a-parameter-to-use-storage-instead-of-memory/123692#123692)

## Example

You can convert implicitly `storage` to `memory` and `calldata` to `memory`, but the reverse is not possible.

```php
string public stor = "banana";

function test(string calldata calld) external view {
    string memory memo = "pizza";

    foo(memo, stor);
    foo(calld, stor);  // Creates a copy of calld in memory and passes as parameter
    foo(stor, stor);   // Creates a copy of stor in memory and passes as parameter
    
    foo(memo, stor);
    // foo(memo, memo); // Cannot convert from memory to storage
    // foo(memo, calld); // Cannot convert from calldata to storage
}

function foo(string memory, string storage) internal view {
    
}

/* This fails because you can only use calldata on external functions
function bar(string calldata) internal view {
    
} */
```

### Other notes on reference:

* Reference types don’t necessarily fit into 32bytes — 256 bits.
* Amount of gas consumed during execution depends on data location. Creating independent copies from reference types are expensive thus it is recommended that mostly inside functions we should choose working with memory data location.
* We must be careful in scenario when two or more different variables point same data location since any change in one variable will impact the others.

### calldata vs memory

**Calldata is almost the same as memory but you cannot manipulate (change) the variable. It's also more gas efficient.**

When using `(string`` `<mark style="color:red;">`memory`</mark>` ``name)`, a **copy of the parameter** is passed into `delete holder[name]` , as per the function below:

```solidity
    function release(string calldata name) public {
        require(holder[name] == msg.sender, "Not your name!");
        delete holder[name];
        emit Release(msg.sender, name);
    }
```

However, using `(string`` `<mark style="color:red;">`calldata`</mark>` ``name)`, the parameter would be passed directly into `delete holder[name]` , without making a copy, thereby saving gas.

```solidity
    function release(string memory name) public {
        require(holder[name] == msg.sender, "Not your name!");
        delete holder[name];
        emit Release(msg.sender, name);
    }
```

Since in the above function, the function parameter name is **not modified** within the scope of the function, we can opt to use calldata instead.&#x20;

No point in **making a fresh copy in a new memory location** (which is what setting it to 'memory') does.

{% hint style="warning" %}
Use memory if you want to be able to **manipulate** the values and calldata when the parameter remain immutable.
{% endhint %}

One gotcha on this is that if you pass in a string (or bytes) from a different internal function where you had just created that string in memory, then you can't use calldata here. Example:

```solidity
contract X {
 function sendString() internal {
    string memory y = "Raja is cool";
    callFunction(y);
 }

//ERROR: because calldata is only used from external calls, 
//and y was created in memory before
 function callFunction(string calldata y) {}  
}
```

Non-modifiable, non-persistent data location **where function arguments are stored,** \


### Why is calldata cheaper?

`calldata` are only parameters of a function which is declared as **external**, which **value is allocated by the caller**, that's why it's gas cost is lower.&#x20;

For this reason only parameters of an external function can be declared as `calldata` (no variables declared inside the function and no parameters of a function which is not external)



### Additional reading

* [https://medium.com/coinmonks/solidity-storage-vs-memory-vs-calldata-8c7e8c38bce](https://medium.com/coinmonks/solidity-storage-vs-memory-vs-calldata-8c7e8c38bce)
* [https://ethereum.stackexchange.com/questions/74442/when-should-i-use-calldata-and-when-should-i-use-memory](https://ethereum.stackexchange.com/questions/74442/when-should-i-use-calldata-and-when-should-i-use-memory)
* [https://ethereum.stackexchange.com/questions/123761/data-location-must-be-memory-or-calldata-for-parameter-in-function-but-none](https://ethereum.stackexchange.com/questions/123761/data-location-must-be-memory-or-calldata-for-parameter-in-function-but-none)
* [https://www.youtube.com/watch?v=wOCIhzAuhgs\&list=PLO5VPQH6OWdVQwpQfw9rZ67O6Pjfo6q-p\&index=34](https://www.youtube.com/watch?v=wOCIhzAuhgs\&list=PLO5VPQH6OWdVQwpQfw9rZ67O6Pjfo6q-p\&index=34)
