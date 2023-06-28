---
description: VRF v1
---

# VRFConsumerBase

## Problem: RNG generation

RNG cannot be done on-chain the blockchain is deterministic -> all nodes must come to the same consensus.

## Solution

Off-chain RNG, but verifiable on-chain.

1. Smart contracts send requests for randomness&#x20;
2. Requests are sent to a VRF Coordinator, which is another contract on-chain
3. The VRF Coordinator hands this request to an off-chain chainlink oracle for RNG.
4. After RNG, VRF Coordinator verifies the randomness of the result on-chain.

![](<../.gitbook/assets/image (66).png>)

## Application

To consume randomness we first need to import and inherit from [`VRFConsumerBase`](https://github.com/smartcontractkit/chainlink/blob/master/contracts/src/v0.8/VRFConsumerBase.sol) and define two required functions:

* `requestRandomness`: Make a request to the VRFCoordinator.
* `fulfillRandomness`: Called by VRFCoordinator when it receives a valid VRF proof.

{% hint style="info" %}
_Your contract should own enough LINK to pay the specified fee._
{% endhint %}

Note, the below values have to be configured correctly for VRF requests to work. You can find the respective values for your network in the [VRF Contracts page](https://docs.chain.link/docs/vrf-contracts/v1).

* `LINK Token` - LINK token address on the corresponding network (Ethereum, Polygon, BSC, etc)
* `VRF Coordinator` - address of the Chainlink VRF Coordinator
* `Key Hash` - public key against which randomness is generated
* `Fee` - fee required to fulfill a VRF request

#### On your contract:

```solidity
import "@chainlink/contracts/src/v0.8/VRFConsumerBase.sol";

contract MyContract is VRFConsumerBase {
    
    bytes32 internal keyHash;
    uint256 internal fee;
    
    uint256 public randomResult;
    
/**
     * Constructor inherits VRFConsumerBase constructor
     * 
     * Network: Kovan
     * Chainlink VRF Coordinator address: 0xdD3782915140c8f3b190B5D67eAc6dc5760C46E9
     * LINK token address:                0xa36085F69e2889c224210F603D836748e7dC0088
     * Key Hash: 0x6c3699283bda56ad74f6b855546325b68d482e983852a7a82979cc4807b641f4
     */
    constructor() 
        VRFConsumerBase(
            0xdD3782915140c8f3b190B5D67eAc6dc5760C46E9, // VRF Coordinator
            0xa36085F69e2889c224210F603D836748e7dC0088  // LINK Token
        )
    {
        keyHash = 0x6c3699283bda56ad74f6b855546325b68d482e983852a7a82979cc4807b641f4;
        fee = 0.1 * 10 ** 18; // 0.1 LINK (Varies by network)
    }
    
    // Requests randomness  
    function getRandomNumber() public returns (bytes32 requestId) {
        require(LINK.balanceOf(address(this)) >= fee, "Not enough LINK - fill contract with faucet");
        return requestRandomness(keyHash, fee);
    }
    // Callback function used by VRF Coordinator
    function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
        randomResult = randomness;
    }
```

MyContract inherits VRFConsumerBase, and therefore its constructor as well.

#### VRFConsumerBase constructor requires two inputs,&#x20;

![VRFConsumerBase constructor](<../.gitbook/assets/image (127).png>)

* \_vrfCoordinator -> address of the VRF coordinator contract on the chain we are deploying to.
* \_link -> address of the Link token contract on the chain.

VRF Coordinator is a smart contract that receives requests, hands them off-chain, and subsequently verifies randomness of the generated number on-chain.

The random number generation is done off-chain via chainlink's oracle network. &#x20;

![](<../.gitbook/assets/image (58).png>)

### Request randomness

```solidity
    function getRandomNumber() public returns (bytes32 requestId) {
        require(LINK.balanceOf(address(this)) >= fee, "Not enough LINK - fill contract with faucet");
        return requestRandomness(keyHash, fee);
    }    
```

In MyContract, getRandomNumber() calls on requestRandomness(keyHash, fee). This has been imported from VRFConsumerbase.sol

* pass the keyhash of oracle node
* pass the fee for request

The return `requestRandomness(keyHash, fee)` **will emit a log** to the chainlink oracle we have specified with keyHash.

The oracle will look for this request, generate a random number. This is then returned on-chain by VRF coordinator.

{% hint style="info" %}
seed is deprecated and no longer used. It is kept for backward-compatibility. See VRFConsumerBase.sol for notes.
{% endhint %}

{% hint style="warning" %}
In your implementation of getRandomNumber() or whichever function that calls on requestRandomness, make sure your contract holds the necessary LINK to successfully call requestRandomness. Else gas estimation error will occur.&#x20;
{% endhint %}

### fulfillRandomness

Callback function used by VRF Coordinator to return the RNG after on-chain verification.&#x20;

```solidity
function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
    randomResult = randomness;
}
```

VRF will call this, by passing the requestID and randomness. We will catch the RNG and store it into `randomResult`.

Techincally, VRFConsumerBase calls `rawFulfillRandomness()` on verification: &#x20;

![results in fulfillRandomness being called](<../.gitbook/assets/image (196).png>)

This ensures that only VRF coordinator can respond and call `fulfillRandomness`, to prevent spoofed responses.&#x20;

Also fulfillRandomness is internal, therefore can only be called by `rawFulfillRandomness,` and not other external functions.

{% hint style="info" %}
VRF Coordinator looks for the function signature of fulfillRandomness on our contract and calls it. Which is why we must inherit this function from VRFConsumerBase and override it to suit our needs.

\*\* Function signature is the hash of the function name.
{% endhint %}

## requestRandomness() mechanics

`requestRandomness()` will call `transferAndCall()` through the LINK interface&#x20;

![VRFConsumerBase.sol](<../.gitbook/assets/image (143).png>)

![LinkTokenInterface.sol](<../.gitbook/assets/image (265).png>)

transferAndCall() originates from the LINK token contract, and is interacted with through an interface that was imported: LinkTokenInterface.sol

{% hint style="info" %}
in VRFConsumerBase.sol \
&#x20;    import "./interfaces/LinkTokenInterface.sol"; \
&#x20;    (state) LinkTokenInterface immutable internal LINK \
&#x20;    (constructor) LINK = LinkTokenInterface(\_link);
{% endhint %}

&#x20;transferAndCall sends our link tokens as fee and will call on the coordinator we specified.

## Function execution breakdown

* Smart contracts send requests for randomness&#x20;
* Requests sent to VRF Coordinator contract, via `requestRandomness(keyHash, fee, userprovidedSeed)` inherited from VRFConsumerBase.sol
*
* VRF Coordinator hands this request to an off-chain chainlink oracle for RNG.
* After RNG, VRF Coordinator verifies the randomness of the result on-chain.
