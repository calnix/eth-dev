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

### **Example 1**: mixed-size array

Here, the fixed-size 2 refers to the **second level of nesting**, not the first.

* \[ \[1,2], \[3,4], \[5,6], \[7,8], ..... ]

```solidity
string[2][ ] crypto_names;
```

```solidity
contract CryptoNames {
    string[2][ ] public crypto_names;

    constructor() public {
        crypto_names.push([“Alice”, “Bob”]);   
        crypto_names.push([“Carol”, “Dave”]);  
        crypto_names.push([“Eve”, “Frank”]);   
        crypto_names.push([“Grace”, “Heidi”]); 
    }
}
```

* crypto\_names is a dynamically sized array, containing elements that are fixed-sized arrays, length 2.
* cannot **crypto\_names.push(\["Ivan", "Judy", "Mallory"]);** as its a fixed-array with 3 elements.

### Example 2: mixed-size array

Fixed-size applied to the top level of nesting.&#x20;

* \[ \[0,1], \[2], \[3], \[4], \[5], \[6], \[7,8,9] ]

```solidity
string[ ][6] names_A_to_F;
```

There can **only** be 6 top level elements, where each element is a dynamic array.&#x20;

```solidity
contract NamesAlphabet {

    string[][6] public names_A_to_F;

    constructor() public {
        names_A_to_F[0] = ["Alice"];
        names_A_to_F[1] = ["Bob"];
        names_A_to_F[2] = ["Carol", "Carlos", "Charlie", "Chuck", "Craig"];
        names_A_to_F[3] = ["Dan", "Dave", "David"];
        names_A_to_F[4] = ["Erin", "Eve"];
    }
```

* Can add as many nested elements as neeeded. no restrictions.

### Example 3: nested fixed

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
