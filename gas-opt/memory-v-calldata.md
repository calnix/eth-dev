# Memory v calldata

## Memory

Used for storing variables that are only needed temporarily during the execution of a function, and cannot be accessed outside the function.

* can be used for both declaring function parameters and within function logic
* mutable (variable values are modifiable)
* non-persistent (the value does not persist after the transaction has completed)

{% hint style="info" %}
More on memory keyword: [https://stackoverflow.com/questions/33839154/in-ethereum-solidity-what-is-the-purpose-of-the-memory-keyword](https://stackoverflow.com/questions/33839154/in-ethereum-solidity-what-is-the-purpose-of-the-memory-keyword)
{% endhint %}

## Calldata

Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.&#x20;

* used to hold function arguments passed in from an external caller; user or another contract.
* immutable: calldata is read-only and cannot be modified by the function.
* calldata is cheaper than memory.

### memory vs calldata

```solidity
function doSome(bytes calldata data) external {}    // uses less gas

function doSome(bytes memory data) external {}
```

* By declaring `calldata`, you can avoid the overhead of copying data into `memory` and reduce the amount of gas needed to execute the function. \[`CALLDATALOAD]`
* `memory` is more expensive than `calldata`. Because, it will copy the data from calldata into local memory, as an additional step. \[`CALLDATACOPY]`

<figure><img src="../.gitbook/assets/image (331).png" alt=""><figcaption></figcaption></figure>

### Why bother using memory then?

<pre class="language-solidity"><code class="lang-solidity"><strong>// this will not compile
</strong>function testImmutable(bytes calldata myBytes) external {
    myBytes[0] = 0xab;
}
</code></pre>

* We cannot modify `calldata`; read-only
* If you need to modify function arguments that are stored in `calldata`, you must first copy them into `memory`.
* So in this scenario, we are better served declaring `myBytes` as `memory`

**Alternatively**

<pre class="language-solidity"><code class="lang-solidity"><strong>// this is more gas intensive 
</strong>function testCallData(bytes calldata data) external {
        bytes memory localData = data;     //create a mutable copy in memory
        localData[0] = 0xab;
}
<strong>
</strong><strong>// this is less gas intensive
</strong>function testMemory(bytes memory data) external {
        data[0] = 0xab;
}
</code></pre>

* memory can be cheaper than calldata if you need to do modifications to the external data passed to the function.
* use `calldata` when you are not going to modify the value of the variable passed as `calldata` inside the function.

{% hint style="info" %}
**If you only need to read the data, you can save some gas by storing it in **_**calldata**_**.**
{% endhint %}

## More on calldata

* `calldata` is where data from <mark style="color:red;">**external**</mark> calls to functions are stored.&#x20;
* `calldata` contains parameters of a function as allocated by the external caller
* `msg.data` of an external call is held in `calldata`
* A byte of calldata costs either 4 gas (if it is zero) or 16 gas (if it is any other value).

{% hint style="info" %}
In external calls to functions (when passing parameter **string**), opt to use **`calldata`**` ``string`, instead of <mark style="color:red;">**`memory`**</mark>` ``string`.
{% endhint %}

### Gotcha!

One gotcha on this is that if you pass in a string (or bytes) from an <mark style="color:red;">**internal**</mark> function, where you had just created that string in memory, then you can't use `calldata` here.&#x20;

**Example**

{% code lineNumbers="true" %}
```solidity
contract X {

 function sendString() internal {
    string memory y = "Calnix is cool";
    callFunction(y);
 }

// ERROR: because calldata is only used from external calls, 
// and y was created in memory before
 function callFunction(string calldata y) {
   // ....do something 
 }  
 
}
```
{% endcode %}

* when execution moves from line 4 to 5, it is not an **external call.** There is no `msg.data` that holds `calldata` to be passed into `callFunction`
* **so in the absence of msg.data, there is no calldata that can be accessed.**&#x20;

### Why is calldata cheaper?

calldata are only parameters of a function which is declared as external, which value is allocated by the caller, that's why it's gas cost is lower.

### EXCEPTION: Layer-2

* In the context of a rollups/layer-2, calldata goes from the cheapest resource to the most-expensive resource.\


### Links

* [https://ethereum.stackexchange.com/questions/74442/when-should-i-use-calldata-and-when-should-i-use-memory/74443#74443](https://ethereum.stackexchange.com/questions/74442/when-should-i-use-calldata-and-when-should-i-use-memory/74443#74443)
* [https://stackoverflow.com/questions/33839154/in-ethereum-solidity-what-is-the-purpose-of-the-memory-keyword](https://stackoverflow.com/questions/33839154/in-ethereum-solidity-what-is-the-purpose-of-the-memory-keyword)
* [https://betterprogramming.pub/solidity-tutorial-all-about-calldata-aebbe998a5fc](https://betterprogramming.pub/solidity-tutorial-all-about-calldata-aebbe998a5fc)
