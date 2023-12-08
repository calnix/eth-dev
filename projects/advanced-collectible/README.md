# Advanced Collectible

## Objective:

Create NFT contract that allows a user to mint a Doggie NFT of random breed. Randomness will be supplied by Chainlink VRF.&#x20;

* Inherit ERC721, VRFConsumerBase&#x20;
* Breed {PUG, SHIBA\_INU, ST\_BERNARD}

### For reference, Simple Collectible:

![](<../../.gitbook/assets/image (325).png>)

### Constructor and Initial layout

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.6.6;

// using OpenZepplin 3.x 
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@chainlink/contracts/src/v0.6/VRFConsumerBase.sol";

contract AdvancedCollectible is ERC721, VRFConsumerBase {
    uint256 public tokenCounter;      
    uint256 public fee;
    uint256 public keyhash;
    
    enum Breed {PUG, SHIBA_INU, ST_BERNARD}
    
    constructor(
    address _vrfCoordinator, 
    address _linkToken, 
    uint _fee, 
    bytes32 _keyhash) public ERC721("Doggie", "DOG"), VRFConsumerBase(_vrfCoordinator, _linkToken) {

        tokenCounter = 0;
        fee = _fee;
        keyhash = _keyhash;
    }

    function createCollectible(string memory tokenURI) public returns(bytes32) {
        bytes32 requestId = requestRandomness(keyhash,fee);  //catch requestID 
    }

    function fulfillRandomness(bytes32 requestId, uint randomness) internal override {
        require(randomness > 0, "RNG not received");

    }

}
```

#### Constructors

We have two nested constructors, each from our inherited contracts. If you choose to parameterize the arguments for the nested constructor, you will have to pass them into the top-level constructor and then again into the nested - like so in VRFConsumerBase above.&#x20;

With ERC721(), both name and symbol are going to be static, so we opt to pass them in directly.

#### Approach: createCollectible & fulfillRandomness

When a user calls `createCollectible`, a request for randomness is initiated to the VRF Coordinator. This request is subsequently passed off-chain to the oracle network for RNG, and back to the VRF Coordinator for on-chain verification.&#x20;

On verification, VRF Coordinator calls the `fulfillRandomness` function on our contract, returning the random value.

Since `fulfillRandomess` receives the random number, we will have to house the breed selection and minting components within this function - different from SimpleCollectible.sol

#### createCollectible()

We want the user that called createCollectible() to be assigned the tokenId and breed generated from the RNG.

To capture this, we use a mapping between requestId and address.

```solidity
function createCollectible(string memory tokenURI) public returns(bytes32) {
    bytes32 requestId = requestRandomness(keyhash,fee);     //catch requestID to associate RNG with user with tokenID
    requestIdtoSender[requestId] = msg.sender;
}
```

#### fulfillRandomness()

```solidity
function fulfillRandomness(bytes32 requestId, uint randomness) internal override {
    require(randomness > 0, "RNG not received");
    Breed breed = Breed(randomness % 3);
    uint mintingTokenId = tokenCounter;
    tokenIdToBreed[mintingTokenId] = breed;
    
    //_safeMint(msg.sender, mintingTokenId);
    
    tokenCounter = ++ tokenCounter;
    return mintingTokenId;  
}
```

* use this to generate a random breed selection -> modulo 3 (since 3 breed types)
* tokenCounter is current available tokenID -> grab it as mintingTokenId
* map mintingTokenId to the random breed attribute selected:

```solidity
// add to declaration section
mapping (uint256 => Breed) public tokenIdToBreed; 
```

* increment tokenCounter and return mintingTokenId (which was just minted)

{% hint style="info" %}
NFT attributes can be associated to their tokenID via mappings
{% endhint %}

**mint NFT -> safeMint()**

{% hint style="warning" %}
cannot use `_safeMint(msg.sender, mintingTokenId)`as we did before.&#x20;

The caller of fulfillRandomness is VRF Coordinator, therefore msg.sender would yield that contract's address.
{% endhint %}

We need to get the original msg.sender of `createCollectible`

This is why the mapping `requestIdtoSender` was introduced earlier. It associates the original caller (user), with their requestId - with this we can extract the correct address. Like so:

```solidity
function fulfillRandomness(bytes32 requestId, uint randomness) internal override {
    require(randomness > 0, "RNG not received");
    Breed breed = Breed(randomness % 3);
    uint mintingTokenId = tokenCounter;
    tokenIdToBreed[mintingTokenId] = breed;
    
    //_safeMint(msg.sender, newTokenId);
    address owner = requestIdtoSender[requestId];
    _safeMint(owner, mintingTokenId);

    tokenCounter = ++ tokenCounter;
    return mintingTokenId;
}
```

## setTokenURI

```solidity
function setTokenURI(uint tokenId, string memory tokenURI) returns() {
    // associate on-chain metadata (breed) with off-chain metadata (URI)
    require(_isApprovedOrOwner(_msgSender(), tokenId), " ERC721: caller is not owner nor approved!");
    _setTokenURI(tokenId, _tokenURI);
}
```

![From ERC721](<../../.gitbook/assets/image (283).png>)

For a token that exists, it returns whether 'spender' is allowed to manage 'tokenId'.\_msgSender()

* \_msgSender() is from Context.sol which ERC721 inherits.

\*\*&#x20;



## Mapping Updates should emit events

This is a recommended best practice.

```solidity
event requestCollectible(bytes32 indexed requestId, address requester);
event breedAssigned(uint indexed tokenId, Breed breed);
```

{% tabs %}
{% tab title="createCollectible" %}
```solidity
function createCollectible() public returns(bytes32) {
    bytes32 requestId = requestRandomness(keyhash, fee);     //catch requestID to associate RNG with user with tokenID
    requestIdtoSender[requestId] = msg.sender;
    emit requestCollectible(requestId, msg.sender);
}
```
{% endtab %}

{% tab title="fulfillRandomness" %}
```solidity
function fulfillRandomness(bytes32 requestId, uint randomness) internal override {
    require(randomness > 0, "RNG not received");
    Breed breed = Breed(randomness % 3);
    uint mintingTokenId = tokenCounter;
    
    tokenIdToBreed[mintingTokenId] = breed;
    emit breedAssigned(mintingTokenId, breed);
    
    //_safeMint(msg.sender, newTokenId);
    address owner = requestIdtoSender[requestId];
    _safeMint(owner, mintingTokenId);
    // setTokenURI
    tokenCounter = ++ tokenCounter;
}
```
{% endtab %}
{% endtabs %}
