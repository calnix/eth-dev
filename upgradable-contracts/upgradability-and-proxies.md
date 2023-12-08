# Upgradability & Proxies

## How does upgradability work?

<figure><img src="../.gitbook/assets/image (359).png" alt=""><figcaption></figcaption></figure>

* A proxy contract exists between the main implementation contract and the user.&#x20;
* The proxy contract serves as the entrypoint of interaction for users; it forwards transactions to the current implementation contract that contains the logic.
* Proxy contract remains unchanged and always has the same address, but the implementation contract that the Proxy contract refers to can be changed. In this way, it can be considered "Upgradeable".

{% hint style="info" %}
* Proxies forward users' transaction via `delegatecall`, so the state is preserved on the proxy contract and the logic executed is retreived from the implementation contract.
* This way data is not lost, even if the referenced implementation contract is changed.
{% endhint %}

## How proxies work

**Proxy contract**

* contract that acts as a proxy, delegating all calls to the contract it is the proxy for
* serves as the storage component

**Implementation Contract**

* Contract that contains the execution logic (state var, functions, etc)
* Contract that you want to upgrade

The proxy contract stores the address of the implementation contract as a state variable.

All user calls go through the proxy and this proxy delegates the calls to the implementation contract at the address that proxy has stored, returning any data it received from logic layer to the caller or reverting for errors.

{% hint style="info" %}
Only borrow the logic from implementation contract and execute it in proxy's context affecting proxy's state variables in storage.
{% endhint %}

### Storage collision between Proxy and Implementation contract

One cannot just go around and simply declare `address implementation` in a proxy contract because that would cause storage collision with the storage of implementation which may have multiple variables in it at overlapping storage slots.

<figure><img src="../.gitbook/assets/image (360).png" alt=""><figcaption></figcaption></figure>

Solution is to write the `implementation` address into a pseudo-random slot. That slot position should be sufficiently random so that having a variable in implementation contract at same slot is negligible.

<figure><img src="../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

According to [EIP-1967](https://eips.ethereum.org/EIPS/eip-1967) one such slot could be calculated as:

```solidity
bytes32 private constant implementationPosition = bytes32(uint256(
  keccak256('eip1967.proxy.implementation')) - 1
));
```

Every time implementation address needs to be accessed/modified this slot is read/written.

{% hint style="info" %}
[EIP-1967](https://eips.ethereum.org/EIPS/eip-1967): A consistent location where proxies store the address of the logic contract they delegate to, as well as other proxy-specific information.
{% endhint %}

### Storage collision between different Implementation version contracts

When upgrading to a new implementation, if a new state variable is added to implementation contract, it MUST be appended in storage layout. The new contract **MUST extend the storage layout and NOT modify it**. Otherwise, collisions may occur.

* this can be done via inheritance

<figure><img src="../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure>

## Initializing constructor code

Any initialization logic should run inside the proxy, as it preserves state. So having a constructor on the implementation contract is nullified as the constructor would assign values to state variables that are held within the context of the implementation contract - not the proxy.

To that end, we have an `initialize` function on the implementation that achieves the same end result as the constructor would, but is delegateCalled by the proxy.

<figure><img src="../.gitbook/assets/image (363).png" alt=""><figcaption></figcaption></figure>

This is just like a normal function but MUST be ensured that it is called only once; hence the `initializer` modifier.&#x20;

### Function clashes between Proxy and Implementation

The proxy contract may have functions of its own - this depends on the proxy pattern employed. In the case of transparent proxies, they contain the function, for example `upgradeTo(address impl)`.&#x20;

What if the implementation contract has a function with the same name i.e. `upgradeTo(address someAddr)`?There must be a mechanism to determine whether to delegate the call to implementation or not.&#x20;

One such way ([OpenZeppelin](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies#transparent-proxies-and-function-clashes) way) is by having an admin or owner address of the Proxy contract.&#x20;

* If the admin (i.e. `msg.sender` == `admin`) is making the calls to Proxy, it will not delegate the call but instead execute the function in Proxy itself, if it exists or reverts.&#x20;
* For any other address it simply delegates the call to implementation.&#x20;
* Thus, only an admin address can call `upgradeTo(address impl)` of the Proxy to upgrade to new version of implementation contract.

## Transparent Proxy (TPP) <a href="#transparent-proxy-tpp" id="transparent-proxy-tpp"></a>

Transparent proxy pattern includes the upgrade functionality within the proxy contract itself. An admin role is assigned with privilege to interact with the proxy contract directly to update the referenced logic implementation address. Callers that do not have the admin privilege will have their call delegated to the implementation contract.

* Implementation address - Located in a unique storage slot in the proxy contract ([EIP-1967](https://proxies.yacademy.dev/pages/proxies-list/#eip-1967-upgradeable-proxy)).
* Upgrade logic - Located in the proxy contract with use of a [modifier](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/transparent/TransparentUpgradeableProxy.sol#L45-L51) to re-route non-admin callers.
* If the caller is the admin of the proxy, the proxy will not delegate any calls,&#x20;
* If the caller is any other address, the proxy will always delegate the call, even if the func sig matches one of the proxy’s own functions.&#x20;

<figure><img src="../.gitbook/assets/image (364).png" alt=""><figcaption></figcaption></figure>

This pattern is very widely used for its upgradeability and protections against certain function and storage collision vulnerabilities.

<details>

<summary>Pro vs Con</summary>

#### Pros <a href="#pros-4" id="pros-4"></a>

* Eliminates possibility of function clashing for admins, since they are never redirected to the implementation contract.
* Since the upgrade logic lives on the proxy, if a proxy is left in an uninitialized state or if the implementation contract is selfdestructed, then the implementation can still be set to a new address.
* Reduces risk of storage collisions from use of [EIP-1967](https://proxies.yacademy.dev/pages/proxies-list/#eip-1967-upgradeable-proxy) storage slots.
* Block explorer compatibility.

#### Cons <a href="#cons-4" id="cons-4"></a>

* Every call not only incurs runtime gas cost of `delegatecall` from the [Proxy](https://proxies.yacademy.dev/pages/proxies-list/#the-proxy) but also incurs cost of SLOAD for checking whether the caller is admin.
* Because the upgrade logic lives on the proxy, there is more bytecode so the deploy costs are higher.

#### Known vulnerabilities <a href="#known-vulnerabilities-4" id="known-vulnerabilities-4"></a>

* [Delegatecall and selfdestruct not allowed in implementation](https://github.com/YAcademy-Residents/Solidity-Proxy-Playground/tree/main/src/delegatecall\_with\_selfdestruct/UUPS\_selfdestruct)
* [Uninitialized proxy](https://github.com/YAcademy-Residents/Solidity-Proxy-Playground/tree/main/src/uninitialized/UUPS\_Uninitialized)
* [Storage collision](https://github.com/YAcademy-Residents/Solidity-Proxy-Playground/tree/main/src/storage\_collision)

</details>

#### Implementation

Re-routing is often implemented with a modifier like this one from [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/transparent/TransparentUpgradeableProxy.sol#L45-L51):

```solidity
modifier ifAdmin() {
    if (msg.sender == _getAdmin()) {
        _;
    } else {
        _fallback(); // redirects call to proxy
    }
}
```

and a check in the `fallback()`:

{% code overflow="wrap" %}
```solidity
require(msg.sender != _getAdmin(), "TransparentUpgradeableProxy: admin cannot fallback to proxy target");
```
{% endcode %}

{% hint style="info" %}
Note: The proxy admin should NOT be any important role or even a regular user for the logic implementation contract because the proxy administrator **cannot interact** with the implementation contract.
{% endhint %}

## UUPS (Universal Upgradeable Proxy Standard)&#x20;

The UUPS pattern was first documented in [EIP1822](https://eips.ethereum.org/EIPS/eip-1822), which describes a standard for an upgradeable proxy pattern where the `upgrade` logic is stored in the implementation contract.&#x20;

* This way, there is no need to check if the caller is admin in the proxy at the proxy level, saving gas.&#x20;
* It also eliminates the possibility of a function on the implementation contract colliding with the upgrade logic in the proxy.
* Because the upgrade mechanism is in the implementation, later versions can remove related logic to disable future upgrades.&#x20;
* Implementation address - Located in a unique storage slot in the proxy contract ([EIP-1967](https://proxies.yacademy.dev/pages/proxies-list/#eip-1967-upgradeable-proxy)).

<figure><img src="../.gitbook/assets/image (365).png" alt=""><figcaption></figcaption></figure>

The upgrade mechanism contains an additional check when upgrading that ensures the new implementation contract is upgradeable. The UUPS proxy contract usually incorporates [EIP-1967](https://proxies.yacademy.dev/pages/proxies-list/#eip-1967-upgradeable-proxy).

<details>

<summary>Pros vs Cons</summary>

#### Pros <a href="#pros-5" id="pros-5"></a>

* Eliminates risk of functions on the implementation contract colliding with the proxy contract since the upgrade logic lives on the implementation contract and there is no logic on the proxy besides the `fallback()` which delegatecalls to the impl contract.
* Reduced runtime gas over TPP because the proxy does not need to check if the caller is admin.
* Reduced cost of deploying a new proxy because the proxy only contains no logic besides the `fallback()`.
* Reduces risk of storage collisions from use of [EIP-1967](https://proxies.yacademy.dev/pages/proxies-list/#eip-1967-upgradeable-proxy) storage slots.
* Block explorer compatibility.

#### Cons <a href="#cons-5" id="cons-5"></a>

* Because the upgrade logic lives on the implementation contract, extra care must be taken to ensure the implementation contract cannot `selfdestruct` or get left in a bad state due to an improper initialization. If the impl contract gets borked then the proxy cannot be saved.
* Still incurs cost of `delegatecall` from the [Proxy](https://proxies.yacademy.dev/pages/proxies-list/#the-proxy).

#### Known vulnerabilities <a href="#known-vulnerabilities-5" id="known-vulnerabilities-5"></a>

* [Uninitialized proxy](https://github.com/YAcademy-Residents/Solidity-Proxy-Playground/tree/main/src/uninitialized/UUPS\_Uninitialized)
* [Function clashing](https://github.com/YAcademy-Residents/Solidity-Proxy-Playground/tree/main/src/function\_clashing)
* [Selfdestruct](https://github.com/YAcademy-Residents/Solidity-Proxy-Playground/tree/main/src/delegatecall\_with\_selfdestruct/UUPS\_selfdestruct)

</details>

#### **Implementation**

You can make the any implementation contract UUPS compliant by making it inherit a common standard interface that requires one to include the upgrade logic, like inheriting OpenZeppelin's [UUPSUpgradeable](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable1) interface.

{% hint style="info" %}
There is little difference between Transparent and UUPS patterns, in the sense that these share the same interface for upgrades and delegation to implementation contract.

The difference lies in where the upgrade logic resides - Proxy or the Implementation contract.
{% endhint %}

{% hint style="success" %}
#### Example <a href="#transparent-vs-uups" id="transparent-vs-uups"></a>

UUPS implementation of upgradable staking pool: [https://github.com/calnix/Upgradable-Staking-Pool](https://github.com/calnix/Upgradable-Staking-Pool)
{% endhint %}

### Transparent vs UUPS Proxies <a href="#transparent-vs-uups" id="transparent-vs-uups"></a>

**Transparent proxies are implemented using** [**`TransparentUpgradeableProxy`**](https://docs.openzeppelin.com/contracts/4.x/api/proxy#TransparentUpgradeableProxy)

Transparent proxies include the upgrade and admin logic in the proxy itself. This means [`TransparentUpgradeableProxy`](https://docs.openzeppelin.com/contracts/4.x/api/proxy#TransparentUpgradeableProxy) is more expensive to deploy than what is possible with UUPS proxies.

**UUPS proxies are implemented using an** [**`ERC1967Proxy`**](https://docs.openzeppelin.com/contracts/4.x/api/proxy#ERC1967Proxy)

* This proxy is not by itself upgradeable.&#x20;
* It is the role of the implementation to include, alongside the contract’s logic, all the code necessary to update the implementation’s address that is stored at a specific slot in the proxy’s storage space.&#x20;
* This is where the [`UUPSUpgradeable`](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable) contract comes in. Inheriting from it (and overriding the [`_authorizeUpgrade`](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable-\_authorizeUpgrade-address-) function with the relevant access control mechanism) will turn your implementation contract into a UUPS compliant implementation.

{% hint style="danger" %}
Note that since both proxies use the same storage slot for the implementation address, using a UUPS compliant implementation with a [`TransparentUpgradeableProxy`](https://docs.openzeppelin.com/contracts/4.x/api/proxy#TransparentUpgradeableProxy) might allow non-admins to perform upgrade operations.
{% endhint %}

## Beacon

The Beacon proxy pattern allows multiple proxy contracts to share one logic implementation by referencing the beacon contract. The beacon contract provides the logic implementation contract address to calling proxies and only the beacon contract needs to be updated when upgrading with a new logic implementation address.

<figure><img src="../.gitbook/assets/image (366).png" alt=""><figcaption></figcaption></figure>

## Diamond Proxy

Diamond Proxy allows us to delegate calls to more than one implementation contract, known as facets, similar to microservices. Function signatures are mapped to facets.

<figure><img src="../.gitbook/assets/image (367).png" alt=""><figcaption></figcaption></figure>

```solidity
mapping(bytes4 => address) facets;
```

Code of call delegation is very similar to the ones that UUPS and Transparent Proxies using but before delegating the call we need to find the correct facet address:

```solidity
// Find facet for function that is called and execute the
// function if a facet is found and return any value.
fallback() external payable {
  // get facet from function selector
  address facet = selectorTofacet[msg.sig];
  require(facet != address(0));
  // Execute external function from facet using delegatecall and return any value.
  assembly {
    // copy function selector and any arguments
    calldatacopy(0, 0, calldatasize())
    // execute function call using the facet
    let result := delegatecall(gas(), facet, 0, calldatasize(), 0, 0)
    // get any return value
    returndatacopy(0, 0, returndatasize())
    // return any return value or error back to the caller
    switch result
      case 0 {revert(0, returndatasize())}
      default {return (0, returndatasize())}
  }
}
```

Benefits of Diamond Proxy:

* All smart contracts have a 24kb size limit. That might be a limitation for large contracts, we can solve that by splitting functions to multiple facets.
* Allows us to upgrade small parts of the contracts (a facet) without having to upgrade the whole implementation.
* Instead of redeploying contracts each time, splitted code logics can be reused across different Diamonds.
* Acts as an API Gateway and allows us to use functionality from a single address.
