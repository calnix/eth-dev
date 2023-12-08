# Brownie Advanced

## Importing packages - Dependencies

```
import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";
import "@chainlink/contracts/src/v0.6/vendor/SafeMathChainlink.sol";
```

Remix knows that these are npm packages to be downloaded.&#x20;

Brownie cannot download directly from npm, but it can download directly from github.

To allow for this, we need to input dependencies into brownie-config.yaml file:

{% code title=" brownie-config.yaml " %}
```python
dependencies:
  # - <organization/repo>@<version>
  #https://github.com/smartcontractkit/chainlink-brownie-contracts
  - smartcontractkit/chainlink-brownie-contracts@0.4.0

compiler:
  solc:
    remappings:
    - '@chainlink=smartcontractkit/chainlink-brownie-contracts@0.4.0'

```
{% endcode %}

* dependencies section tells brownie what to download from where.

![](<../../../.gitbook/assets/image (258).png>)

* compiler section tells compiler that @chainlink is to be remapped to a specific dependency that was downloaded.&#x20;
* will be downloaded in to `build/contracts/dependencies` as a json

{% hint style="info" %}
if you get file outside of allowed directories or not found, might need to:

npm install @chainlink/contracts OR npm install @chainlink/contracts --save\
\
run cmds in project root directory&#x20;
{% endhint %}
