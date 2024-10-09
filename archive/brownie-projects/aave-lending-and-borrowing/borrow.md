# Borrow

We need to find out how much we can borrow against our deposits/collateral.

## getUserAccountData()

* [getUserAccountData](https://docs.aave.com/developers/v/2.0/the-core-protocol/lendingpool#getuseraccountdata)
* **`function getUserAccountData(address user)`**
* values are returned in WEI, we will convert to ETH

```python
def get_borrowable_data(lending_pool,account):
    (totalCollateralETH, totalDebtETH, availableBorrowsETH, currentLiquidationThreshold, ltv, healthFactor) = lending_pool.getUserAccountData(account)
    
    # all ETH values are denominated in WEI - convert to ETH
    total_collateral_eth = Web3.fromWei(totalCollateralETH, "ether")
    total_debt_eth = Web3.fromWei(totalDebtETH, "ether")
    available_borrow_ETH = Web3.fromWei(availableBorrowsETH, "ether")

    print(f"...You have {total_collateral_eth} worth of ETH deposited...")
    print(f"...You have {total_debt_eth} worth of ETH borrowed...")
    print(f"...You can borrow {available_borrow_ETH} worth of ETH...")
    return (float(available_borrow_ETH), float(total_debt_eth))
```

* return multiple variables using tuple syntax.
* float allows for decimal places.&#x20;

### Into main()

add the following code

```python
# Borrow DAI
## how much can we borrow - pull account stats - getUserAccountData(address user)
(borrowable_eth, total_debt_eth) = get_borrowable_data(lending_pool,account)
```

## DAI

We now know much we can borrow in terms of ETH, {available\_borrow\_ETH}, from above.

How much is that in terms of DAI?&#x20;

We will need to get DAI/ETH price and convert -> via chainlink pricefeed contract&#x20;

### Price of DAI/ETH

Chainlink has a standard interface contract AggregatorV3Interface which works with all of its different price feed contracts.

With this interface, we simply need to pass the corresponding price feed address into it, to create the contract object in brownie.

```python
def get_token_price(token_price_feed_address):
    # use chainlink pricefeed
    token_eth_price_feed = interface.AggregatorV3Interface(token_price_feed_address)
    price = token_eth_price_feed.latestRoundData()[1]   #price is index 1 
    norm_price = Web3.fromWei(price, "ether")
    print(f"...DAI/ETH price is {norm_price}")
    return float(price)
```

* create price feed contract object
* calls latestRoundData from price feed contract, slicing out price variable from return tuple
* convert price from Wei to Ether (decimal place modification)

### Into main()

```python
# Borrow DAI
## how much can we borrow - pull account stats - getUserAccountData(address user)
(borrowable_eth, total_debt_eth) = get_borrowable_data(lending_pool,account)

# DAI in terms of ETH
dai_eth_price = get_token_price(config["networks"][network.show_active()]["dai_eth_price_feed"])
```

## Lending\_pool.borrow

#### call borrow() from lendingPool contract

```
## function borrow(address asset, uint256 amount, uint256 interestRateMode, uint16 referralCode, address onBehalfOf)
# interestRateMode: 1 - stable | 2 - variable
tx2 = lending_pool.borrow(config["networks"][network.show_active()]["dai_token_address"], dai_to_borrow, 1, 0, account.address, {"from": account})
```

We will need to add the DAI token address into brownie-config.yaml

* mainnet-fork: get DAI address off etherscan
* Kovan/Rinkeby: refer to Aave documentation to get their testnet version of DAI&#x20;

{% hint style="warning" %}
For assets on testnets, we use different versions of the token (e.g. testnet Dai). This is to ensure enough liquidity for our reserves and to easily mint more tokens when needed.

If you are developing on a testnet and require tokens, go to [https://testnet.aave.com/faucet,](https://testnet.aave.com/faucet) making sure that your wallet is set to the relevant testnet.
{% endhint %}

* [https://docs.aave.com/developers/deployed-contracts/v3-testnet-addresses](https://docs.aave.com/developers/deployed-contracts/v3-testnet-addresses)
* [https://docs.aave.com/developers/v/2.0/deployed-contracts/deployed-contracts](https://docs.aave.com/developers/v/2.0/deployed-contracts/deployed-contracts)

**Testnet assets addresses are different depending on stable vs variable rate of borrowing**

