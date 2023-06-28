# Fixed-point Math

## Solidity supports integers but not decimals

Solidity scales up decimals into integers, to sidestep its ability to offer floats. The number of decimals in ERC20 determines the **factor by which decimals are scaled up.**

For instance, 18 means that the value is stored in an uint as `decimalValue * 10**18`

* 1 unit of token is represented as 1e18.
* 10 units of token is represented as 10e18.
* 0.5 units of token is represented as 5e17.
* 100.103 => stored as **uint** 100103000000000000000  _<mark style="background-color:yellow;">(100.103 \*10\*\*18)</mark>_
* 0.000340 => stored as uint 340000000000000  _<mark style="background-color:yellow;">(0.00034\*10\*\*18)</mark>_

### Converting 1 DAI to WETH (18 dp -> 18 dp)

Both DAI and WETH are of 18 decimal precision. Let us assume 1 DAI = 0.00039472576 ether.

Exchange rate on contract => DAI/ETH: 394725760000000 (0.00039472576 \* 1e18)&#x20;

**Conversion**

* 1 DAI \* DAI/ETH = Qty of Ether
* `1e18` DAI x `394725760000000` = `394725760000000...0`_(18 more '0's)_
* `1e18` x \[`0.00039472576` \* `1e18`] = `0.00039472576`` `<mark style="background-color:red;">`*1e18 *1e18`</mark>

When multiplying two integers, and the resulting value is **scaled up twice**. Which is why we **divide**, to remove the extra scaling factor, **after** multiplication.

* `1e18` x \[`394725760000000`] / decimal\_precision&#x20;
* `1e18` x \[`394725760000000`] / `1e18` = `394725760000000`&#x20;
* `394725760000000` represents `0.00039472576`&#x20;

### **Converting between different decimal representations**

* TokenA is set up with 18 decimals
* TokenB is set up with 6 decimals

If you want to convert between different representations you have to multiply or divide depending on the difference in decimals count:

```markdown
To convert valueA to valueB: (18 dp -> 6 dp)
    valueB = valueA / 10**(A.decimals - B.decimals)
    valueB = valueA / (10**(18-6))
```

**Normalize** the _**from\_token**_ by stripping it of its decimal places.&#x20;

![](<../../.gitbook/assets/image (95).png>)

Then we scale up by the TokenB (to token) decimals

### Vice versa

```markdown
**To convert valueB to valueA: (6 dp -> 18 dp)
    valueA = valueB * 10**(B.decimals - A.decimals)
    valueA = valueB * (10**(18-6))

    valueA = (valueB / 10**6) * 10**18
```

![](<../../.gitbook/assets/image (28).png>)
