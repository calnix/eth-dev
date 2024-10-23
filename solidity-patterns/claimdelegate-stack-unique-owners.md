# claimDelegate: stack unique owners

* for some array of tokenIds: \[0,1,2,3,...]
* the owners array, could contain repeated owners: \[userA, userB, userA, userA]
* this collapses the owner array to unique owners, and calculates their total claimable amount to be transferred out
* the uniqueArrays have the same length of the original arrays -- they have trailing 0s to fill up the replaced elements

```solidity
    /**
     * @notice Users to claim via delegated hot wallets
     * @dev msg.sender is designated delegate of nfts
     * @param tokenIds Nfts' tokenId
     */  
    function claimDelegated(uint256[] calldata tokenIds) external whenStartedAndBeforeDeadline whenNotPaused {
        
        // array validation
        uint256 tokenIdsLength = tokenIds.length;
        if(tokenIdsLength == 0) revert EmptyArray(); 

        // check delegation on msg.sender
        bytes[] memory data = new bytes[](tokenIdsLength);
        address[] memory owners = new address[](tokenIdsLength);
        for (uint256 i = 0; i < tokenIdsLength; ++i) {
            
            uint256 tokenId = tokenIds[i];

            // get and store nft Owner
            address nftOwner = NFT.ownerOf(tokenId);          
            owners[i] = nftOwner;

            // data for multicall
            data[i] = abi.encodeCall(IDelegateRegistry(DELEGATE_REGISTRY).checkDelegateForERC721, 
                        (msg.sender, nftOwner, address(NFT), tokenId, ""));
        }
        
        // data for staticCall
        bytes memory staticData = abi.encodeCall(IDelegateRegistry(DELEGATE_REGISTRY).multicall, data); 
        
        // staticCall
        (bool success, bytes memory result) = DELEGATE_REGISTRY.staticcall(staticData); 
        if (!success) revert StaticCallFailed();

        // if a tokenId is not delegated will return false; as a bool
        bytes[] memory results = abi.decode(result, (bytes[]));


        //note: ending w/ 0s as placeholder
        address[] memory uniqueOwners = new address[](tokenIdsLength);
        uint256[] memory uniqueAmounts = new uint256[](tokenIdsLength);

        uint256 totalAmount;
        uint256 uniqueCounter;

        for (uint256 i = 0; i < tokenIdsLength; ++i) {
            
            // multiCall uses delegateCall: decode return data
            bool isDelegated = abi.decode(results[i], (bool));
            if(!isDelegated) revert InvalidDelegate();

            // update tokenId: storage is updated
            uint256 tokenId = tokenIds[i];
            uint256 claimable = _updateLastClaimed(tokenId);
            
            totalAmount += claimable;

            address owner = owners[i];
            
            // populate 1st element
            if(i == 0) {
                
                uniqueOwners[i] = owner;
                uniqueAmounts[i] = claimable;

                ++uniqueCounter;

            } else {    
                
                // check all prev.| reverse loop decrement to 0
                for (uint256 j = i-1; j >= 0; j--) {
                    
                    // check if match w/ any of the previous owners
                    if (owner == uniqueOwners[j]) {

                        uniqueAmounts[j] += claimable;
                        break;
                    }
                    
                    // If we've reached the beginning and found no match
                    if (j == 0) {
                        
                        // add to array
                        uniqueOwners[i] = owner;
                        uniqueAmounts[i] = claimable;

                        ++uniqueCounter;
                    }
                }

            }
        }
```

