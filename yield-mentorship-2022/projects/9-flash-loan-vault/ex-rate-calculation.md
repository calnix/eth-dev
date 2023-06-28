# Ex-rate calculation

Since we start with a peg, and need the exchange rate to float, the logic differs from #4 Fractional Wrapper implementation

### getExchangeRate()

```solidity
    /// @notice Returns the unit 'exchange rate'; assuming 1 unit of underlying was deposited, how much shares would be received 
    /// @dev wrapperMinted = (underlyingDeposited,(1) * wrapperSupply) / underlyingInWrapper 
    /// Note: Exchange rate is floating, it's dynamic based on capital in/out-flows
    /// Note: _totalSupply returns a value extended over decimal precision, in this case 18 dp. hence the scaling before divsion.
    function getExchangeRate() internal view returns(uint256) {
        uint256 sharesMinted;
        return _totalSupply == 0 ? sharesMinted = 1e18 : sharesMinted = (_totalSupply * 1e18) / underlying.balanceOf(address(this));
    }

    /// @notice Calculate how much yvDAI user should get based on flaoting exchange rate
    /// @dev getExchangeRate() returns shares minted per unit of underlying asset deposited; at present moment.
    /// @param assets Amount of underlying tokens (assets) to be converted to wrapped tokens (shares)
    function convertToShares(uint256 assets) internal view returns(uint256 shares){
        return assets * getExchangeRate() / 1e18;
    }

    /// @notice Calculate how much DAI user should get based on floating exchange rate
    /// @dev getExchangeRate() returns shares minted per unit of underlying asset deposited; at present moment.
    /// @param shares Amount of wrapped tokens (shares) to be converted to underlying tokens (assets) 
    function convertToAssets(uint256 shares) internal view returns(uint256 assets){
        return shares * 1e18 / getExchangeRate();
    }
```

Since we want a currency peg of 1:1 to be in effect when \_totalSupply is 0, getExchangeRate() returns 1 (1 unit = 1e18).&#x20;

If\_totalSupply is a non-zero value, exchange rate is calculated based on proportionality:

```
sharesMinted = (_totalSupply * 1e18) / underlying.balanceOf(address(this))
```

It is important to ensure sharesMinted is raised to the power of 18, so as to keep in-line with the decimal precision of the numbers we will be interacting with.

This also ensures that we do not end up with a decimal value for sharesMinted.

### totalSupply

According to the [ERC20 standard](https://theethereum.wiki/w/index.php/ERC20\_Token\_Standard) `totalSupply` returns an `uint`. Hence, this should be the amount of tokens in the smallest unit your token offers. This is not in Wei or any Ether unit, this is just the number of tokens (in your smallest unit) which you decide that exist.

Still, your token can specify decimals. For example, if you set `uint8 public constant decimals = 8;`, your token would support 8 decimal places. But this is only for convenience, i.e. `totalSupply` still needs to return values in the smallest unit.

For example, let's say you offer 1000 tokens with 2 decimal places. Consequently, `totalSupply` needs to return `100000` (i.e. `1000 * 100`).

{% hint style="info" %}
tokenSupply = tokensIActuallyWant \* (10 \*\* decimals)

If I want 1 token with the ability to subdivide it with a precision of 2 decimal places, I represent that as 100.
{% endhint %}
