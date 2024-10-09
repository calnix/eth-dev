# Dependencies: import contracts

```solidity
// Get the latest ETH/USD price from chainlink price feed
import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";
import "@chainlink/contracts/src/v0.6/vendor/SafeMathChainlink.sol";  

```

Brownie cannot download from npm, so it doesn't automatically know what to do with @chainlink

Brownie can download form Github.

Tell brownie where in GitHub to download them from using Brownie-config.yaml:

```yaml
dependencies:
  # - <organization/repo>@<version>
  #https://github.com/smartcontractkit/chainlink-brownie-contracts
  - smartcontractkit/chainlink-brownie-contracts@0.4.0

compiler:
  solc:
    remappings:
    - '@chainlink=smartcontractkit/chainlink-brownie-contracts@0.4.0'
```

remapping tells Brownie what to do with the @chainlink prefix, mapping it to the dependency name.

