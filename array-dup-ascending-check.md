# Array: dup/ascending check

### generic dup check

```solidity
// duplicate check
function _checkDuplicateElements(uint256[] memory tokenIds) internal pure returns(uint256) {
    uint256 incomingNfts = tokenIds.length;

    if(incomingNfts > 1) {
        for(uint256 i; i < incomingNfts; ++i) {
            for(uint256 j; j < incomingNfts; ++j) {
               
                if(i == j) continue; // comparing same ele
               
                if(tokenIds[i] == tokenIds[j]) revert Errors.DuplicateIds();
            }
        }
    }

    return incomingNfts;
}
```

### ascending check

* if array does not contain 0

```solidity
uint256 previous;
for (uint256 i; i < array.length; i++){
    if (previous >= array[i]) revert();
    previous = array[i];
}
```

* if can contain 0

```solidity
uint256 previous;
for (uint256 i; i < array.length; i++){
    if (i != 0) {
        if (previous >= array[i]) revert();
    }
    previous = array[i];
}
```
