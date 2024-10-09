# Multi-Dimensional arrays

## Nested/Multi-D arrays

**Multi-dimensional arrays are essentially nested arrays (An array that contain other arrays)**

* T\[k]\[k] : Two-Dimensional, Fixed-size
* T\[ ]\[ ] : Two-Dimensional, Dynamic-size
* T\[ ]\[k] or T\[k]\[] : Two-Dimensional, Mixed-size

**Multi-dimensional arrays can have any level of nesting**

* T\[2]\[2]\[2] : Three-Dimensional, Fixed-Size (all k are the same)
* T\[2]\[8]\[4]\[12] : Four-Dimensional, Fixed-Sizes ( k‘s are of different values)
* T\[ ]\[ ]\[ ]\[ ]\[ ] : Five-Dimensional, Dynamic-Size
* T\[ ]\[3]\[2]\[ ]\[9]\[ ] : Six-Dimensional, Mixed-Size

{% hint style="info" %}
* You cannot have different types within nested arrays
* The maximum level of nesting for nested arrays is 15. If you try to create a variable with 16 nested arrays, you will get a ‘[stack too deep](https://medium.com/coinmonks/stack-too-deep-error-in-solidity-608d1bd6a1ea)’ error. As stack pointer in the EVM cannot access a slot in the stack that is deeper than is 16th element (top down).
{% endhint %}

### Indexing and Referencing

Things may be a little counter-intuitive because the **X & Y axis are reversed.**

```solidity
bool[2][] flags;
```

* top-level is a dynamic array, with any number of elements
* each nested element is a fixed-array of size 2

So, to reference such things, the dynamic dimension is high order.

```csharp
bool flag = flags[dynamicIndex][lengthTwoIndex];
```

```solidity
    uint256[2][] public flags; 

    function test1() public returns(uint256) {
        
        flags.push([1, 2]); //flag[0]
        flags.push([3, 4]);
        flags.push([5, 6]);
        flags.push([7, 8]); //flag[3]

        return flags[3][1]; //returns 8
    }
```

* When declaring, we are declaring in reverse, from bottom to the top-most level.&#x20;
* When slicing/indexing, we operate normally, from top down.

## Mixed-size: nested dynamic

Fixed-size applied to the top level of nesting.  The nested elements can be of any length.

* **\[** \[0,1], \[2], \[3], \[7,8,9] **]**

```solidity
// only 4 top-level elements
uint256[][4] public values;
```

There can **only** be 4 top level elements, where each element is a dynamic array.&#x20;

### Storage&#x20;

```solidity
contract MixedTest{

    // top fixed
    uint256[][4] public values;

    function mixedStorage() public returns(uint256) {
        
        values[0] = [0, 1];     
        values[1] = [2];
        values[2] = [2];
        values[3] = [3,4,5,6];  
        
        //push into nested dynamic array
        values[0].push(13);  

        return values.length; //returns 4
    }
```

* Can add as many nested elements as needed. no restrictions.
* If its a storage array, we can use push to add new elements.
* **values\[0] = \[0, 1, 13]**

{% hint style="warning" %}
Push is only for storage arrays
{% endhint %}

### Memory

* push is not available
* Must init the nested dynamic array with a specified length.
* However, each nested array can have a different length.
* "Free-sizing" arrays are not allowed in memory - their lengths must be defined and unchanged for memory allocation.&#x20;

```solidity
    function mixedMem() public pure returns(uint256) {

        // no "free-sizing" in memory.
        // must init the nested array w/ a specified length.
        uint256[][4] memory values;
        values[0] = new uint256[](2);
        
        //values[0] = [uint256(1), 2] will not work
        values[0][0] = 1;
        values[0][1] = 2;

        //value[1]: len 3
        values[1][0] = 2;
        values[1][1] = 3;
        values[1][2] = 4;
        
        return values[1][2]; // returns 4
    }
```

* values\[0] is of length 2
* values\[1] is of length 3

#### **Use loop to populate elements of a nested array**

```solidity
    function mixedMem2() public pure returns(uint256){
        // nested "dynamic". 4 top.
        uint256[][] memory tree = new uint256[][](4);

        // create memory reference to nested array
        uint256 nestedLength = 3;
        uint256[] memory nodes = tree[0] = new uint256[](nestedLength);
        
        // add elements via reference
        for(uint256 i = 0; i < arrayLength; i++){
            nodes[i] = 6;          
    }
```

* nodes and tree\[0] are connected by the same memory reference
* any changes made to one, is reflected in the other, as the same memory location is being updated.
* **tree\[0] = \[6,6,6] = nodes**

{% hint style="info" %}
* Dynamic nested arrays in memory are possible contingent on the fact that their lengths are defined before assignment.&#x20;
* This reduces them to a fixed-size array.
* However, they are "dynamic" in that each nested array could be of different length.&#x20;
{% endhint %}

## Mixed-size array, top dynamic&#x20;

Top level is dynamic, can take as many elements as needed. However, each element must be an array of size 2.

```solidity
// [ [1,2], [3,4], [5,6], [7,8], ..... ]
uint256[2][ ] values;
```

### Storage&#x20;

```solidity
contract MixedTest2 {

    uint256[2][ ] public values;

    function addElements() public returns(uint256) {
        values.push([1, 2]);   
        values.push([3, 4]);   
        values.push([5, 6]);   
        
        // cannot: values[3] = [7, 8];
        // will revert
        
        return values.length; //3
    }
}
```

* Cannot **values.push(\[3, 4 , 5]);** as its a fixed-array with 3 elements.
* Cannot **values\[3] = \[7, 8]**; must use **push method**.

### Memory

```solidity
    function memoryMixed() public pure returns(uint256) {
        uint256[2][] memory values;

        //top is dynamic: so must define size. 
        values = new uint256[2][](2);

        //2nd ele
        values[1] = [uint256(3), 4];
        
        return values[1][1]; //4
    }
```

* The top level is dynamic and its length is undefined;&#x20;
* We must first define it.&#x20;
* In doing so it becomes a nested fixed-fixed array.

{% hint style="info" %}
This might seems redundant. But who knows?
{% endhint %}

## Nested fixed-fixed

* 2 top-level elements
* each element can be an array of fixed size 4
* \[ \[1,2,3,4], \[5,6,7,8] ]

```solidity
    function test1() public pure returns (uint256) {
        // entirely fixed 
        uint256[4][2] memory myArr;
        
        return myArr[1].length;    //length = 4
    }
```

* ```solidity
  myArr.length = 2
  myArr[1].length = 4
  ```

### Example 4: nested dynamic&#x20;

```solidity
    function test2() public pure returns (uint256) {
        
        bytes32[][] memory tree;
        
        //return tree.length;    //length = 0
        return tree[0].length;    //reverts 

    }
```

* tree.length = 0
* tree\[0].length will revert.

