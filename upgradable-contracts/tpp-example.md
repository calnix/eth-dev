---
description: EIP-1538
---

# TPP Example

## Proxy&#x20;

We can use OpenZepplin's [TransparentUpgradeableProxy](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/transparent/TransparentUpgradeableProxy.sol). The file contains both an interface and the proxy contract.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The explanation states that we are to use the provided interface to interact with the proxy contract, as the ABI procuded by the compiler will be missing some functions due to an internal dispatch mechanism.&#x20;

#### Internal dispatch mechanism

The mechanism is essentially how the fallback function is crafted.&#x20;

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

If the function signature provided in the call matches one of the interface functions, the call is routed to the corresponding internal function.&#x20;

For example, if the proxy admin calls `upgradeTo`, `_dispatchUpgradeTo` is executed as outlined above. All other calls are forwarded to the implementation.&#x20;

Due to the nature of this fallback mechanism, `upgradeTo` is not defined within the proxy contract and hence the ABI will not include it. Therefore, the interface provided is required.

### Proxy Admin

{% hint style="warning" %}
1. If any account other than the admin calls the proxy, the call will be forwarded to the implementation, even if that call matches one of the admin functions exposed by the proxy itself.
2. If the admin calls the proxy, it can access the admin functions, but its calls will never be forwarded to the implementation. If the admin tries to call a function on the implementation it will fail with an error that says "admin cannot fallback to proxy target".
{% endhint %}

**OpenZepplin's recommendation:**

* A dedicated account to be an instance of the {ProxyAdmin} contract. If set up this way, you should think of the `ProxyAdmin` instance as the real administrative interface of your proxy.
* [Proxy Admin contract ](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/transparent/ProxyAdmin.sol)

## Implementation

Nothing special needs to be done on the implementation contract. We could use the same staking pool in the UUPS example - only that we would need to remove the upgradability logic on it.







{% hint style="info" %}
[https://forum.openzeppelin.com/t/initializing-admin-in-the-constructor-of-a-uups-proxy-contract/34232](https://forum.openzeppelin.com/t/initializing-admin-in-the-constructor-of-a-uups-proxy-contract/34232)
{% endhint %}
