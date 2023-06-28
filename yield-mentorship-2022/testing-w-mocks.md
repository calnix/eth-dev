---
description: >-
  https://github.com/yieldprotocol/vault-v2/blob/master/packages/foundry/contracts/test/utils/Mocks.sol
---

# Testing w/ Mocks

For unit testing within Foundry, we rely on Mocks.sol to simulate calls to other contracts (e.g. Ladle, Cauldron) as well as their return values.&#x20;

Mocks.sol is a library and has to be attached to each abstract contract - not just the first, StateZero.

#### **Usage in test contract:**

```solidity
import "./utils/Mocks.sol" 

abstract contract WitchV2StateZero is Test, TestConstants { 
    using Mocks for *;
...
}
```

## **Mocks.sol**

* collection of function signatures&#x20;
* relies on function overloading for ease of use -> can simply call mock on an external method to be simulated.

### mock() functions:

* based on vm.mockCall()
* Mocks a call to an address, returning specified calldata.

![mockCall](<../.gitbook/assets/image (179).png>)

#### Example:

```solidity
// In ContangoLadle.t.sol:
address bob = address(0xb0b);

DataTypes.Vault memory vault = DataTypes.Vault({
            owner: bob,
            seriesId: "series",
            ilkId: "ilk"
        });

cauldron.build.mock(bob, "vaultId", "series", "ilk", vault); 
cauldron.build.verify(bob, "vaultId", "series", "ilk");


// in Cauldron.sol: 
function build(address owner, bytes12 vaultId, bytes6 seriesId, bytes6 ilkId)
```

* mock `build` function in cauldron&#x20;
* since mock is a library fn, it takes the function `build` as its first parameter:

![matching mock fn signature](<../.gitbook/assets/image (31).png>)

*   `mockCall` takes 3 parameters:

    * address of bob -> f.address&#x20;
    * input calldata -> abi.encodeWithSelector(f.selector, p1, p2, p3, p4)
    * returns calldata -> abi.encode(r1)



**We need to specify the data that is returned. In this case, we expect the struct vault to be the returns calldata.**

* hence `vault` as specified when calling mock

## Mocking a contract

![](<../.gitbook/assets/image (320).png>)

**Mocks.mock("Cauldron") matchs the fn signature:**

![Mock.sol](<../.gitbook/assets/image (24).png>)

* StrickMock()

![](<../.gitbook/assets/image (308).png>)

* a placeholder contract is deployed at an address and labelled as "Cauldron".
* the deployed address is returned, which is subsequently passed into the interface ICauldron.

## verify() functions:

* based on `expectCall()` from Vm.sol

![](<../.gitbook/assets/image (155).png>)

* test in general a particular path was hit in a smart contract that calls another contract
