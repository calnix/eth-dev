---
description: EIP-1822
---

# UUPS Example

## Proxy&#x20;

* UUPS proxy is fairly minimal compared to the transparent proxy setup.&#x20;
* Using OZ's implementation library, you don't need to do anything else other than simply deploying the `ERC1967Proxy.sol` contract and linking your implementation.

{% code title="UUPSProxy" %}
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {StakingPoolStorage} from "./StakingPoolStorage.sol";
import {ERC1967Proxy} from "openzeppelin-contracts/contracts/proxy/ERC1967/ERC1967Proxy.sol";

/// @title Proxy for UUPS implementation
/// @author Calnix
contract StakingPoolProxy is ERC1967Proxy {

    /**
     * @dev Initializes the upgradeable proxy with an initial implementation specified by `_logic`.
     *
     * If `_data` is nonempty, it's used as data in a delegate call to `_logic`. This will typically be an encoded
     * function call, and allows initializing the storage of the proxy like a Solidity constructor.
     */
    constructor(address _logic, bytes memory _data) ERC1967Proxy(_logic, _data){
    }


    /// @dev Returns the current implementation address.
    function implementation() external view returns (address impl) {
        return _implementation();
    }
}
```
{% endcode %}

## Implementation

We will need to have our implementation contract inherit `UUPSUpgradeable`, which defines the upgradeability mechanism to be included in the implementation contract.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

import {StakingPoolStorage} from "./StakingPoolStorage.sol";
import {SafeERC20, IERC20} from "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";

import {ERC20Upgradeable} from "openzeppelin-contracts-upgradeable/contracts/token/ERC20/ERC20Upgradeable.sol";
import {OwnableUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/OwnableUpgradeable.sol";
import {UUPSUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/proxy/utils/UUPSUpgradeable.sol";


/// @title A single-sided staking pool that is upgradeable
/// @author Calnix
/// @notice Stake TokenA, earn Token A as rewards
/// @dev Rewards are held in rewards vault, not in the staking pool. Necessary approvals are expected.
/// @dev Contract should be inherited and deployed. See MyStakingPool as example.
/// @dev Pool is only compatible with tokens of 18 dp precision.
contract StakingPoolBase is StakingPoolStorage, ERC20Upgradeable, OwnableUpgradeable, UUPSUpgradeable {
    using SafeERC20 for IERC20;

    /// @notice Initializes the Pool
    /// @dev To be called by proxy on deployment
    /// @param stakedToken Token accepted for staking
    /// @param rewardToken Token emitted as rewards
    /// @param rewardsVault Vault that holds rewards
    /// @param owner Owner of Pool
    /// @param name ERC20 name of staked token (if receive TokenA for staking, mint stkTokenA to user)
    /// @param symbol ERC20 symbol of staked token (if receive TokenA for staking, mint stkTokenA to user)
    function initialize(IERC20 stakedToken, IERC20 rewardToken, address rewardsVault, address owner, string memory name, string memory symbol) 
        external virtual initializer {
            STAKED_TOKEN = stakedToken;
            REWARD_TOKEN = rewardToken;
            REWARDS_VAULT = rewardsVault;

            __ERC20_init(name, symbol);
            __Ownable_init(owner);
            __UUPSUpgradeable_init();
    }
    
    //.... other functions.....
    
    /*//////////////////////////////////////////////////////////////
                             UPGRADEABILITY
    //////////////////////////////////////////////////////////////*/

    ///@dev override _authorizeUpgrade with onlyOwner for UUPS compliant implementation
    function _authorizeUpgrade(address newImplementation) internal onlyOwner override {}
```

Inheriting from `UUPSUpgradeable` (and overriding the [`_authorizeUpgrade`](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable-\_authorizeUpgrade-address-) function with the relevant access control mechanism) will turn your contract into a UUPS compliant implementation.

{% hint style="info" %}
Codebase: [https://github.com/calnix/Upgradable-Staking-Pool](https://github.com/calnix/Upgradable-Staking-Pool)
{% endhint %}

### Disable Initializer

```solidity
constructor() {
    _disableInitializers();
} 
```

`_disableInitializer()` is called inside the constructor; prevents any further initialization. This is done so that in the context of the logic contract the initializer is locked.&#x20;

Therefore an attacker will not able to call the `initalizer()` function in the state of the logic contract and perform any malicious activity (like self-destruct)

* Note that the proxy contract state will still be able to call this function, this is because the constructor does not effect the state of the proxy contract.&#x20;
* When you upgrade your contract you need to follow the convention of inheriting the previous version and then adding any modifications.&#x20;
* It is not expected that you will need to initialize something every time. So, if you shouldn’t confuse this as preventing any upgrades.

{% hint style="info" %}
**Upgradeable smart contracts can have constructors**

* Upgradeable smart contracts cannot have any constructors which initialize any variable data.&#x20;
* Can have a constructor that initializes an immutable (constants are initialized inline).&#x20;
{% endhint %}

{% hint style="warning" %}
Need to use `reinitializer()` in subsequent upgrades for the initializer function
{% endhint %}

{% hint style="info" %}
More info:&#x20;

* [https://abhik.hashnode.dev/6-nuances-of-using-upgradeable-smart-contracts](https://abhik.hashnode.dev/6-nuances-of-using-upgradeable-smart-contracts)
* [https://ethereum.stackexchange.com/questions/132261/initializing-the-implementation-contract-when-using-uups-to-protect-against-atta](https://ethereum.stackexchange.com/questions/132261/initializing-the-implementation-contract-when-using-uups-to-protect-against-atta)
{% endhint %}
