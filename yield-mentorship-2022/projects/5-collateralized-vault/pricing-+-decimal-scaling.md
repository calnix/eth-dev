# Pricing + Decimal scaling

### Solidity cannot handle decimals -> are scaled up to integers

The number of decimals in ERC20 determines the **factor** used to represent the **decimal number as uint**.&#x20;

* For instance 18 means that the value is stored in an uint as `decimalValue * 10**18`.
* 100.103 => is stored as **uint** 100103000000000000000  _<mark style="background-color:yellow;">(100.103 \*10\*\*18)</mark>_
* 0.000340 => stored as uint 340000000000000  _<mark style="background-color:yellow;">(0.00034\*10\*\*18)</mark>_

> <mark style="color:red;background-color:yellow;">**Let us refer to this as the 'solidity decimal scale up effect'**</mark>

### Converting 1 DAI to WETH (18 dp -> 18 dp)

**assume DAI/ETH: 394725760000000**

1e18 x 394725760000000 = 394725760000000...(18 more '0's)

1e18 x 0.00039472576**e18 =** 0.00039472576 <mark style="background-color:red;">\*e18 \*e18</mark>

* \*e18 is the factor to represent the number of decimal places.
* when multiplying two integers (both have <mark style="color:red;background-color:yellow;">**solidity decimal scale up effect**</mark>), and the **scale up effect is applied twice**
* which is why we **divide** by decimals after multiplication.
  * 1e18 x 0.00039472576**e18 =** 0.00039472576 <mark style="background-color:red;">\*e18</mark>

{% hint style="info" %}
this is a result of using the scale up approach, due to the inability of Solidity to handle decimals.
{% endhint %}

### **Converting between different decimal representations**

* TokenA is set up with 18 decimals
* TokenB is set up with 6 decimals

If you want to convert between different representations you have to multiply or divide depending on the difference in decimals count:

```markdown
**To convert valueA to valueB: (18 dp -> 6 dp)
    valueB = valueA / 10**(A.decimals - B.decimals)
    valueB = valueA / (10**(18-6))
```

**Normalize** the _**from\_token**_ by stripping it of its decimal places. \
(because of the <mark style="color:red;background-color:yellow;">**solidity decimal scale up effect**</mark>)

![](<../../../.gitbook/assets/image (95).png>)

Then we scale up by the TokenB (to token) decimals

### Vice versa

```markdown
**To convert valueB to valueA: (6 dp -> 18 dp)
    valueA = valueB * 10**(B.decimals - A.decimals)
    valueA = valueB * (10**(18-6))

    valueA = (valueB / 10**6) * 10**18
```

![](<../../../.gitbook/assets/image (28).png>)

{% hint style="info" %}
naturally division will lead to precision loss.
{% endhint %}

## Weaving pricing into scaling&#x20;

#### getMaxDebt()  | <mark style="color:blue;">how much DAI for WETH?</mark>

<figure><img src="../../../.gitbook/assets/image (62).png" alt=""><figcaption><p>USING WETH COLLATERAL TO BORROW DAI</p></figcaption></figure>

#### getMaxDebt()  | <mark style="color:blue;">how much USDC for WETH?</mark>

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
For two tokens with different decimals precision, the decimal precision of their price would match the token with the larger decimal precision\
\
Example: \
TokenA -> 6 dp \
TokenB -> 18 dp \
\
TokenA /TokenB price is returned as 18 dp; else we would incorrectly truncate the precision of tokenB,&#x20;

**This is why the chainlink pricefeed for USDC/ETH is in 18dp.**
{% endhint %}

<details>

<summary>Old illustration</summary>

![](<../../../.gitbook/assets/image (63).png>)

</details>
