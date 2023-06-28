# Forking and Mocking

FundMe.sol requires chainlink oracle contracts for price feeds on eth/usd. Currently it pulls this data from the contracts deployed on Rinkeby network, by passing the pricing contract's address into an interface (AggregatorV3Interface.sol)

* AggregatorV3Interface priceFeed = AggregatorV3Interface(0x8A753747A1Fa494EC906cE90E9f37563A8AF630e);

{% hint style="info" %}
Testing can be done on live mainnet and testnet - but will be a slow process with mining lag and ETH fees. So we look to test in a private environment free of such constrains -> hence forking or mocking.
{% endhint %}

**What if we want to do testing on our private development chain?** &#x20;

Problem: Chainlink contracts do not exists there.&#x20;

**Forking**

* copy the mainnet (via Alchemy/Infura API)&#x20;
* run mainnet fork in our private env&#x20;
* prior contracts deployed on mainnet wil be available in our forked local env.
* can simply use mainnet address references.&#x20;

**Mocking**

* deploy a mock contracts of the real thing in our development chain.
  * like MockV3Aggregator, where we decide ETH/USD price.
* deploy mock contracts on persistent mock network
* use dummy accounts \[get\_accounts -> account\[0] -> dummy acc]

## On brownie networks

There are 3 kinds of networks brownie connects to:

* `Ethereum`
* `Ethereum Classic` (We can ignore this)
* `Development`

You can see them by running `brownie networks list` in your terminal.

When creating a network in the `Ethereum` category, brownie will save the addresses of the contracts that were deployed, storing addresses in the `build` folder.&#x20;

```python
Ethereum
  ├─Mainnet (Infura): mainnet
  └─ganache-local: ganache (brownie will remember these)

Development
  ├─Ganache-CLI: development
  └─ganache-temp: ganache-temp (brownie won't remember these)
```

ganache-local therefore has persistency - at least till we close the ganacheUI.

If you'd like the desired experience to be such that brownie always redeploys, you can create your ganache network in the `Development` network instead, so brownie won't remember it's deployments, and will always deploy fresh.

This will mean though, that you won't be able to run any of the 2nd scripts, like `brownie run scripts/price_feed_scripts/02_whatever_this_one_is.py` since brownie won't remember any contracts being there.

{% hint style="info" %}
brownie default network: development (Ganache-CLI)
{% endhint %}

{% hint style="warning" %}
brownie automatically latches onto ganacheUI if it is running, regardless of where we specify the --network flag to be.

This can cause errors, especially when you deploying to forked mainnet instances.

**Always consider if ganacheUI has any business running in regards to your network deployment location.**&#x20;
{% endhint %}
