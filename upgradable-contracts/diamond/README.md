# 🚧 Diamond

## Diamond standard

The Diamond standard is a finalized Ethereum Improvement Proposal ([EIP-2535](https://eips.ethereum.org/EIPS/eip-2535)) that aims to make it easier for developers to modularize and upgrade their smart contracts.

The core idea is that you can control many implementation contracts from your single Diamond contract (proxy contract). Some key features of the Diamond standard include:

* A single gateway to make proxy calls to _n_ number of implementation contracts
* Upgrade a single or multiple smart contract atomically
* No storage limit to how many implementation contracts you can add to your Diamond
* A log history of all the upgrades made on the Diamond
* Can reduce gas costs (i.e., by reducing the number of external function calls)

## Diamond

The diamond is the smart contract that serves as the proxy. It calls (delegatecall) other implementation contracts known as facets.&#x20;

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/******************************************************************************\
* Author: Nick Mudge <nick@perfectabstractions.com> (https://twitter.com/mudgen)
* EIP-2535 Diamonds: https://eips.ethereum.org/EIPS/eip-2535
*
* Implementation of a diamond.
/******************************************************************************/

import { LibDiamond } from "./libraries/LibDiamond.sol";
import { IDiamondCut } from "./interfaces/IDiamondCut.sol";

contract Diamond {    

    constructor(address _contractOwner, address _diamondCutFacet) payable {        
        LibDiamond.setContractOwner(_contractOwner);

        // Add the diamondCut external function from the diamondCutFacet
        IDiamondCut.FacetCut[] memory cut = new IDiamondCut.FacetCut[](1);
        bytes4[] memory functionSelectors = new bytes4[](1);
        functionSelectors[0] = IDiamondCut.diamondCut.selector;
        cut[0] = IDiamondCut.FacetCut({
            facetAddress: _diamondCutFacet, 
            action: IDiamondCut.FacetCutAction.Add, 
            functionSelectors: functionSelectors
        });
        LibDiamond.diamondCut(cut, address(0), "");        
    }

    // Find facet for function that is called and execute the
    // function if a facet is found and return any value.
    fallback() external payable {
        LibDiamond.DiamondStorage storage ds;
        bytes32 position = LibDiamond.DIAMOND_STORAGE_POSITION;
        // get diamond storage
        assembly {
            ds.slot := position
        }
        // get facet from function selector
        address facet = ds.selectorToFacetAndPosition[msg.sig].facetAddress;
        require(facet != address(0), "Diamond: Function does not exist");
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
                case 0 {
                    revert(0, returndatasize())
                }
                default {
                    return(0, returndatasize())
                }
        }
    }

    receive() external payable {}
}
```

* [https://github.com/mudgen/diamond-3-hardhat/blob/main/contracts/Diamond.sol](https://github.com/mudgen/diamond-3-hardhat/blob/main/contracts/Diamond.sol)

### Facets

* Implementation contract or lib -> contracts that hold external fn, that the proxy(diamond) will call to.
* No limit on number of facets
* Facets are deployed separately from your Diamond contract and can be added into your Diamond using `diamondCut`

The benefit of Facets within a Diamond contract is that it allows you to modularize your code and only update what's needed. Unlike a typical proxy pattern where you would need to redeploy a new implementation contract, with Facets since you can have many implementation contracts, you can update multiple Facets (e.g., multiple implementation contracts) or an individual Facet.

#### Example

```solidity
contract FacetA {

  function setDataA(bytes32 _dataA) external {
    LibA.DiamondStorage storage ds = LibA.diamondStorage();
    require(ds.owner == msg.sender, "Must be owner.");
    ds.dataA = _dataA;
  }

  function getDataA() external view returns (bytes32) {
    return LibA.diamondStorage().dataA;
  }
}
```
