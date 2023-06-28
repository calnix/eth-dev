# iterate Array, Return Sum

## working with calldata

```solidity
contract Question1 {
    function iterateEachElementInArrayAndReturnTheSum(uint256[] calldata array) external pure returns (uint256)  {
        // TODO: Iterate each element in the array using only assembly
        uint256 sum = 0;

        assembly {

            //locates the length of the string in the calldata and copies it to memory
            let len := calldataload(0x24) 
            
            for {let i := 0} lt(i, len) {i := add(i,1)}
            {
                // free memory pointer 
                let ptr := add(mload(0x40), mul(i, 0x20)) 
                
                // creates a new in-memory copy of the input array
                // calldatacopy(copyToMemoryLocation, copyFromCallDataLocation, copySize)
                //copyFromCallDataLocation: add(0x24, 0x20): location of len+32 bytes, to get first char
                calldatacopy(ptr, add(add(0x24, 0x20), mul(i, 0x20)), 32)
                sum := add(sum, mload(ptr))

            }
        }

        return sum;
    }
}
```

```solidity
/** Explanation:

    First we obtain the length of the array passed: let len := calldataload(0x24)
    
     calldataload(startingOffset) loads 32 bytes starting from the specified offset in the calldata onto the stack.
     
     The first 4 bytes of calldata contain the function signature.
     The next 32 bytes (0x20) in calldata point to the location in calldata where the array begins.
     The subsequent 32 bytes is the length space. (data following the array’s length is the actual array content).
     
     To get length of array, we want an startingOffset of 0x24 (32+4 = 36 bytes).
     Therefore, calldataload(0x24) loads the length of the array.

    Then we set up a for loop to iterate through the array elements. 
    
    Free memory pointer: let ptr := add(mload(0x40), mul(i, 0x20)) 
     mload(0x40) gives us the free memory pointer. 
     Since we need to iterate and load multiple elements in a loop, on each interation we increment the pointer by 32 bytes (0x20).
     1st element will be loaded to memory location: 0x40
     2nd element will be loaded to memory location: 0x40 + 0x20 = 0x60

    calldatacopy(copyToMemoryLocation, copyFromCallDataLocation, copySize) <=> calldatacopy(ptr, add(add(0x24, 0x20), mul(i, 0x20)), 32)
    
     calldatacopy copies the calldata starting from 'copyFromCallDataLocation', for 'copySize', into memory storing it at 'copyToMemoryLocation'
     The 1st element located 32 bytes after the length space, at add(0x24, 0x20) = 0x44
     Each element is of size 32 bytes, hence copySize = 32.
     
     mul(i, 0x20) allows the for loop to traverse down the calldata space, from element to element. 
     
 */
```

* [https://betterprogramming.pub/solidity-tutorial-all-about-calldata-aebbe998a5fc](https://betterprogramming.pub/solidity-tutorial-all-about-calldata-aebbe998a5fc)
* [https://medium.com/swlh/getting-deep-into-evm-how-ethereum-works-backstage-ab6ad9c0d0bf](https://medium.com/swlh/getting-deep-into-evm-how-ethereum-works-backstage-ab6ad9c0d0bf)
* [https://medium.com/@kalexotsu/understanding-solidity-assembly-hashing-a-string-from-calldata-fbd2ece82263](https://medium.com/@kalexotsu/understanding-solidity-assembly-hashing-a-string-from-calldata-fbd2ece82263)

### Inspiration code

```solidity

assembly {

    //locates the length of the string in the calldata and copies it to memory
    // first 4 bytes are function selector - ignore: 0x20+4 = 36 bytes
    // first 32 bytes point to the location in calldata where the byte array begins - 0x20 = 32 bytes
    let len := calldataload(0x24) // stores size of array on init: 5
    
    // free memory pointer 
    let ptr := mload(0x40)
    
    // creates a new in-memory copy of the input array
    // calldatacopy(copyToMemoryLocation, copyFromCallDataLocation, copySize)
    //copyFromCallDataLocation: add(0x24, 0x20): location of len+32 bytes, to get first char
    calldatacopy(ptr, add(0x24, 0x20), 32)
    sum := mload(ptr)
}
```

* this will return the first element in the array

## creates in-memory copy of the input calldata

```solidity
contract Question1 {
    function iterateEachElementInArrayAndReturnTheSum(uint256[] calldata array) external pure returns (uint256)  {
        // TODO: Iterate each element in the array using only assembly
        
        // creates a new in-memory copy of the input array
        uint256[] memory arrayM = array;
        uint256 sum = 0;
        
        for (uint i = 0; i < arrayM.length; ++i) {
            assembly {
                // 0x20 needs to be added to an array because the first slot contains the array length.
                sum := add(sum, mload( add(add(arrayM, 0x20), mul(i, 0x20)) ))
            }
        }

        return sum;
    }
}
```

<details>

<summary>Explanation</summary>

```solidity
/** Explanation:

    for (uint i = 0; i < data.length; ++i) 
     Starts a loop that iterates over each element of the array using the loop variable i. 
     The loop runs as long as i is less than the length of array.

    sum := add(sum, mload( add(add(data, 0x20), mul(i, 0x20)) )) 
      Reads the value of the i-th element of array and adds it to the sum variable
      The outermost add function is used to perform the addition, and the mload function is used to load the i-th element from memory.
    
    How do we get the memory address of the i-th element, for mload?
    add( add(data, 0x20), mul(i, 0x20) ) 
     The add and mul functions are used to calculate memory offsets.
     
     The 0x20 value is added to data to skip over the first 32 bytes (the length field), 
     and then i is multiplied by 0x20 (32 bytes) to calculate the offset of the i-th element (since each element is 32 bytes in size).

*/
```

</details>

```solidity

create a copy of calldata in memory and worh with that.
since the Solidity for loop does not allow direct access to the elements of a calldata array.


for (uint i = 0; i < arrayM.length; ++i) 
 starts a loop that iterates over each element of the arrayM copy using the loop variable i. 
 The loop runs as long as i is less than the length of arrayM.

assembly { sum := add(sum, mload(add(add(arrayM, 0x20), mul(i, 0x20)))) } 
  reads the value of the i-th element of arrayM and adds it to the sum variable

  The outermost add function is used to perform the addition, and the mload function is used to read a 256-bit word from memory.

add(add(arrayM, 0x20), mul(i, 0x20)) 
 The add and mul functions are used to calculate memory offsets.
 The add and mul functions are used to calculate the memory address of the i-th element of the array. 
 The 0x20 value is added to arrayM to skip over the first 32 bytes (the length field), 
 and then i is multiplied by 0x20 to calculate the offset of the i-th element (since each element is 32 bytes in size).
```

## References

* [https://docs.soliditylang.org/en/v0.8.17/assembly.html](https://docs.soliditylang.org/en/v0.8.17/assembly.html)
* [https://blog.openzeppelin.com/ethereum-in-depth-part-2-6339cf6bddb9/](https://blog.openzeppelin.com/ethereum-in-depth-part-2-6339cf6bddb9/)

#### Incorrect inspiration

````solidity
```solidity
        for (uint i = 0; i < array.length; ++i) {
            assembly {
                let a := add(mload(0x40), mul(i, 0x20))     //memory pointer for 1st var
                calldatacopy(a, add(4, mul(i, 0x20)), 32)  //to store the first parameter in memeory location, a
                sum := add(sum, mload(a))

            }
        }
``` 
````
