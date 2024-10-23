# claimDelegate: cache previous user

* expects tokenIds to be ordered by the Front-end
* ordered such that, same owners are sequence&#x20;
* since this has a 1 element lookback to compare is the owner matches

```solidity
    /**
     * @notice Users to claim via delegated hot wallets
     * @dev Expects tokenIds to be ordered based on common ownership: [ownerA, ownerA, ownerB]
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

        uint256 totalAmount;
        uint256[] memory amounts = new uint256[](tokenIdsLength);

        address addressCache; 
        uint256 amountCache;

        for (uint256 i = 0; i < tokenIdsLength; ++i) {
            
            // multiCall uses delegateCall: decode return data
            bool isDelegated = abi.decode(results[i], (bool));
            if(!isDelegated) revert InvalidDelegate();

            // update tokenId: storage is updated
            uint256 tokenId = tokenIds[i];
            uint256 claimable = _updateLastClaimed(tokenId);
            
            totalAmount += claimable;
            amounts[i] = claimable;

            // initial reference
            if (i == 0) {

                addressCache = owners[i];
                amountCache = claimable;

            } else { 

                // check owner matches previous tokenid's owner
                if (addressCache == owners[i]) {
                    // increment amountCache
                    amountCache += claimable; 

                } else {  // if different owner from previous token id

                    // transfer current amountCache
                    TOKEN.safeTransfer(addressCache, amountCache); 

                    // update cache to current token id info
                    addressCache = owners[i];
                    amountCache = claimable;
                } 
            }
        }
       

        if (amountCache != 0) {
            TOKEN.safeTransfer(addressCache, amountCache); 
        }

        // update totalClaimed
        totalClaimed += totalAmount;

        // claimed per tokenId
        emit ClaimedByDelegate(msg.sender, owners, tokenIds, amounts);
    }
```

