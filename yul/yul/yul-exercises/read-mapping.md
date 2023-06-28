---
description: '// TODO: Read the mapping value without calling ownerOf[tokenId]'
---

# read mapping

{% tabs %}
{% tab title="Correct" %}
```solidity
contract Question3 {
    mapping(uint256 => address) private ownerOf;

    constructor() {
        ownerOf[0] = address(1);
        ownerOf[1] = address(2);
        ownerOf[2] = address(3);
        ownerOf[3] = address(4);
        ownerOf[4] = address(5);
    }

    function readOwnerOf(uint256 tokenId) external view returns (address) {
        // TODO: Read the mapping value without calling ownerOf[tokenId]
        uint256 slot;
        address ret;

        assembly {
            slot := ownerOf.slot
        }

        bytes32 location = keccak256(abi.encode(tokenId, uint256(slot)));

        assembly {
            ret := sload(location)
        }

        return ret;
    }
}
```
{% endtab %}

{% tab title="Wrong?" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract Question3 {
    mapping(uint256 => address) private ownerOf;

    constructor() {
        ownerOf[0] = address(1);
        ownerOf[1] = address(2);
        ownerOf[2] = address(3);
        ownerOf[3] = address(4);
        ownerOf[4] = address(5);
    }

    function readOwnerOf(uint256 tokenId) external view returns (address result) {
        // TODO: Read the mapping value without calling ownerOf[tokenId]

        assembly{
            // Store tokenId in memory scratch space
            mstore(0, tokenId)
            // Store slot number in scratch space after tokenId
            mstore(32, 0)
            // Create hash from previously stored num and slot
            let hash := keccak256(0, 64)
            // Load mapping value using the just calculated hash
            result := sload(hash)
        }
    }
}
```
{% endtab %}
{% endtabs %}
