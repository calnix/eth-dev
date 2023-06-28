# Interfaces: IERC20

Typical implementation of interface, where we **pass an address into an interface object.**

```solidity
IERC20 public immutable wmdToken;    

constructor(address wmdToken_){
        wmdToken = IERC20(wmdToken_);
    }
```

* can make the parameter of the constructor as `IERC20` this will save the explicit conversion that you are doing.

```solidity
IERC20 public immutable wmdToken;    

///@param wmdToken_ contract address
    constructor(IERC20 wmdToken_){
        wmdToken = wmdToken_;
    }
```

### <mark style="color:yellow;">Why</mark>

By passing an address into an interface, it serves as a pointer to the actual contract allowing us to call the functions declared within the interface.

* _since compiling the interface will provide us with the function signatures of the contract -  which are needed to make function calls._

However, the point here is that the parameter is only considered an **IERC20 in the scope of the compiler.** Once the bytecode is generated, **the parameter will be an address** for purposes of type checking.

Meaning:

* before compilation -> IERC20 wmdToken\_ is treated of type **IERC20.**
* **after compilation ->** IERC20 wmdToken\_ will be compiled as => IERC20(address(wmdToken))
  * wmdToken = wmdToken\_; then simply becomes the assignment like so:
    * wmdToken = ERC20(address(wmdToken))

{% hint style="danger" %}
If you have another contract that calls this constructor, and you compile both together, **they both need to use IERC20**.&#x20;

If you compile them separately and deploy them, one can use IERC20 and the other an address.
{% endhint %}
