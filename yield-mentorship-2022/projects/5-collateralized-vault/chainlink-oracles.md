# Chainlink Oracles

If you are ever unsure what the price is returned by the oracle, just test in remix

![DAi/WETH pricefeed returns](<../../../.gitbook/assets/image (89).png>)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// imports from npm chainlink package
 import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";


contract CheckPrice {
      
    AggregatorV3Interface priceFeed;

    constructor() {
        priceFeed = AggregatorV3Interface(0x74825DbC8BF76CC4e9494d0ecB210f676Efa001D);// dai/eth address on rinkeby
    }

    function getVer() public view returns(uint){
        return priceFeed.version();
    }

    function getPrice() public view returns(uint){
        (,int256 answer,,,) = priceFeed.latestRoundData();
        return uint(answer);  
    }

    function decimals() external view returns (uint8){
        AggregatorV3Interface priceFeed = AggregatorV3Interface(0x74825DbC8BF76CC4e9494d0ecB210f676Efa001D);
        return priceFeed.decimals();
    }
}
```

{% hint style="info" %}
Chainlink will inform you the decimals at:

[https://docs.chain.link/docs/ethereum-addresses/](https://docs.chain.link/docs/ethereum-addresses/)
{% endhint %}

[https://ethereum.stackexchange.com/questions/118809/solidity-chainlink-aggregatorv3interface-decimals-returns-number-8-always](https://ethereum.stackexchange.com/questions/118809/solidity-chainlink-aggregatorv3interface-decimals-returns-number-8-always)
