# UniswapV2Library

pairFor -> calculates the CREATE2 address of the liquidity pool

* getReserves → fetches and sorts the reserves for a pair
* getAmountsOut -> Iterates over the path of exchanges needed and performs chained getAmountOut calculations on any number of pairs
* getAmountOut → given an input amount of an asset and pair reserves, returns the maximum output amount of the other asset considering a 0.3% fee

### sortTokens

```solidity
// returns sorted token addresses, used to handle return values from pairs sorted in this order
function sortTokens(address tokenA, address tokenB) internal pure returns (address token0, address token1) {
    require(tokenA != tokenB, "UniswapV2Library: IDENTICAL_ADDRESSES");
    (token0, token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
    require(token0 != address(0), "UniswapV2Library: ZERO_ADDRESS");
}
```

If the tokens weren't sorted, it would result in a distinct pair address for each sort order:

* Tokens: \[A,B] -> Pair address: X
* Tokens: \[B,A] -> Pair address: Y

By sorting the tokens first, the address of any given pair is pre-determined, and can be computed without any on-chain lookups ([using CREATE2](https://github.com/Uniswap/uniswap-v2-core/blob/master/contracts/UniswapV2Factory.sol#L32)).

{% hint style="info" %}
You have to sort the tokens first, otherwise you will get 2 different pair addresses for each pair.
{% endhint %}

Regardless of scenario, token0 will be the smaller address:

{% tabs %}
{% tab title="First Tab" %}
<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>first address always smallest</p></figcaption></figure>
{% endtab %}

{% tab title="Code" %}
```solidity
console.log("scenario 1:sortTokens(address(1), address(2))");
(address token0, address token1) = UniswapV2Library.sortTokens(address(1), address(2));
console.log("token0:", token0);
console.log("token1:", token1);

console.log("scenario 2:sortTokens(address(2), address(1))");
(address token2, address token3) = UniswapV2Library.sortTokens(address(2), address(1));
console.log("token0:", token2);
console.log("token1:", token3);


```
{% endtab %}
{% endtabs %}

### pairFor

compute pair (liquidity pool) addresses _without any on-chain lookups_ because of [CREATE2](https://eips.ethereum.org/EIPS/eip-1014). The following values are required for this technique:

<table data-header-hidden><thead><tr><th width="281"></th><th></th></tr></thead><tbody><tr><td><code>address</code> </td><td>The <a href="https://docs.uniswap.org/contracts/v2/reference/smart-contracts/factory#address">factory address</a></td></tr><tr><td><code>salt</code></td><td><code>keccak256(abi.encodePacked(token0, token1))</code></td></tr><tr><td><code>keccak256(init_code)</code></td><td><code>0x96e8ac4277198ff8b6f785478aa9a39f403cb768dd02cbee326c3e7da348845f</code></td></tr></tbody></table>

* `token0` must be strictly less than `token1` by sort order.
* `UniswapV2Factory` is deployed at `0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f`
* factory creates liquidity pools via `createPair`

{% hint style="info" %}
[https://docs.uniswap.org/contracts/v2/guides/smart-contract-integration/getting-pair-addresses](https://docs.uniswap.org/contracts/v2/guides/smart-contract-integration/getting-pair-addresses)
{% endhint %}
