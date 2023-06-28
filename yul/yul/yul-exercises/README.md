# Yul Exercises

## In-memory

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

```solidity
WORKING: https://docs.soliditylang.org/en/v0.8.17/assembly.html
    function iterateEachElementInArrayAndReturnTheSum(uint256[] calldata array) external pure returns (uint256)  {
        // TODO: Iterate each element in the array using only assembly
        
        uint256[] memory arrayM = array;
        uint256 sum = 0;

        for (uint i = 0; i < arrayM.length; ++i) {
            assembly {
                sum := add(sum, mload(add(add(arrayM, 0x20), mul(i, 0x20))))
            }
        }

        return sum;
    }

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

* [https://blog.openzeppelin.com/ethereum-in-depth-part-2-6339cf6bddb9/](https://blog.openzeppelin.com/ethereum-in-depth-part-2-6339cf6bddb9/)

## working with calldata

```solidity
contract Question1 {
    function iterateEachElementInArrayAndReturnTheSum(uint256[] calldata array) external pure returns (uint256 sum) {
        // TODO: Iterate each element in the array using only assembly
        assembly {
           
            // len := calldataload(0x24)
            for {let i := 0} lt(i, calldataload(0x24)) {i := add(i,1)}
            {        
                // calldatacopy(copyToMemoryLocation, copyFromCallDataLocation, copySize)
                // calldatacopy(ptr, add(add(0x24, 0x20), mul(i, 0x20)), 32)
                sum := add(sum, calldataload(add(add(0x24, 0x20), mul(i, 0x20))))
            }
        }
    }
}
```

* Since the array is passed as calldata, I looked to work will calldata as much as possible, instead of memory.
* **To iterate, we need the length of the array**
  * We obtain the length of the array with calldataload(0x24).&#x20;
  * calldataload(startingOffset) loads 32 bytes starting from the specified offset in the calldata onto the stack.&#x20;
  * Instead of using 'let len := calldataload(0x24)', we reference calldataload(0x24) directly into the for loop to save on gas.
* **On calldataload(0x24):**
  * The first 4 bytes of calldata contain the function signature.
  * The next 32 bytes (0x20) in calldata point to the location in calldata where the array begins.
  * The subsequent 32 bytes is the length space. (data following the array’s length is the actual array content).
* ```solidity
       To get length of array, we want an startingOffset of 0x24 (32+4 = 36 bytes).
       Therefore, calldataload(0x24) loads the length of the array.
  ```
* Then we set up a for loop to iterate through the array elements.
* ```solidity
  calldataload(add(add(0x24, 0x20), mul(i, 0x20)))
  ```
  * 1st element is located 32 bytes after the length space, at add(0x24, 0x20) = 0x44
  * 2nd element is located 32 bytes after the first element, at add(add(0x24, 0x20), 32)
  * So to traverse down the calldata space, from element to element in the loop, we add mul(i, 0x20) to the 1st element's position
  * Essentially, mul(i, 0x20) allows for iteration of elements in the array by increasing the memory offset from the 1st element
