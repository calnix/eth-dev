# Arrays

**Can be fixed or dynamic sized at compile-time**

* T\[k] -> fixed size of type T, with k elements
* T\[] -> dynamic size of type T
* T\[]\[5] -> 5 dynamic-sized arrays of T (read in reversed)

## **Storage arrays**

**Can be fixed or dynamic.**&#x20;

**For fixed:**&#x20;

* size must be specified during declaration.&#x20;
* Once declared, the array size cannot be changed.&#x20;
* Any attempt to reference beyond the last array position will result in an error.

**For dynamic:**

* size of the array is not mentioned during the declaration.
* size of the dynamic array can be changed during the runtime as elements are added/removed.

### **Init arrays**

**Fixed arrays:**

* all elements initialize to 0  ->  **fixedArr1\[1] = 0**
* can partially assign, as seen in `initializeFixedArr2()`. **fixedArr2 = \[6,0,0]**
* length is 3, as elements are initialized to default values.

**Dynamic arrays:**

* have not defined no. of elements.
* so cannot call dynArr\[0]. will revert.
* must init w/ values first.
* default length of 0.&#x20;

```solidity
contract Array {

    // Several ways to initialize an array
    uint[] public dynArr;
    uint[] public dynArr2 = [1, 2, 3];    

    // Fixed sized array, all elements initialize to 0
    uint[3] public fixedArr1;
    uint[3] public fixedArr2;
    uint[3] public x = [8, 15, 32];

    // FIXED

    function updateFixedArr1() public {
        fixedArr1 = [21, 232, 43];
    }

    function updateFixedArr2() public {
        fixedArr2[0] = 6;
    }

    function getFixedLength() public view returns (uint) {
        return fixedArr1.length;
    }

    // DYNAMIC

    // will fail if dynArr has not been assigned any elements.
    // also, not needed, since dynArr is public.
    function get(uint i) public view returns (uint) {
        return dynArr[i];
    }

    function getLength() public view returns (uint) {
        return dynArr.length;
    }
}
```

* When calling **`get`** on **`dynArr`** it will revert, as no assignment has been done.
* Can call `length` on an unassigned array, it will return 0.

### **Methods**

* length
* delete
* push/pop (only for dynamic)

In a fixed-size array, we can not insert a new element as it increases the length of an array. To insert a new element in the dynamic size array we use the push method; this increases the length.

{% hint style="info" %}
#### **push():** append a zero-initialized element

#### **push(X):** append X&#x20;
{% endhint %}

```solidity
contract Array {

    // Several ways to initialize an array
    uint256[] public dynArr;
 
    // will fail is dynArr has not been assigned any elements.
    // also, not needed, since dynArr is public.
    function get(uint i) public view returns (uint) {
        return dynArr[i];
    }

    // Append to array: increases length by 1.
    function push() public {
        dynArr.push(11);
        dynArr.push(12);
        dynArr.push(13);
    }
    
    // Remove last element from array: decreases length by 1
    function pop() public {
        dynArr.pop();
    }

    function getLength() public view returns (uint) {
        return dynArr.length;
    }

    // Delete does not change the array length.
    // It resets the value at index to it's default value,
    function remove(uint index) public {

        delete dynArr[index];
    }
}
```

**Push and Pop**

* `dynArr` being initialized but unassigned.
* call `push()`: this assigns 3 elements to it. its length is now 3.&#x20;
* `dynArr[2]`: returns 13
* call pop(): removes last element, 13. length is now 2.&#x20;
  * dynArr\[2]: will revert. there is no element there anymore.&#x20;
  * dynArr\[1]: returns 12

**Delete**

* call push()  ->  dynArr = \[11, **12**, 13]
* delete dynArr\[1]  ->  dynArr = \[11, **0**, 13]
* length remains 3

### Removing a specific element

1. **Remove array element by shifting elements from right to left**

Copy all the elements forward, starting from the specified index. Pop the last element as its repeated.

```solidity
contract ArrayRemoveByShifting {

    // [1, 2, 3] -- remove(1) --> [1, 3, 3] --> [1, 3]
    // [1, 2, 3, 4, 5, 6] -- remove(2) --> [1, 2, 4, 5, 6, 6] --> [1, 2, 4, 5, 6]
    // [1, 2, 3, 4, 5, 6] -- remove(0) --> [2, 3, 4, 5, 6, 6] --> [2, 3, 4, 5, 6]
    // [1] -- remove(0) --> [1] --> []

    uint[] public arr;

    function remove(uint _index) public {
        require(_index < arr.length, "index out of bound");

        // copy all the elements forward, starting from the specified index.
        // pop the last element as its repeated.
        // [1, 2, 3] -- remove(1) --> [1, 3, 3] --> [1, 3]
        for (uint i = _index; i < arr.length - 1; i++) {
            arr[i] = arr[i + 1];
        }
        arr.pop();
    }

    function test() external {
        arr = [1, 2, 3, 4, 5];
        remove(2);
        // [1, 2, 4, 5]
        assert(arr[0] == 1);
        assert(arr[1] == 2);
        assert(arr[2] == 4);
        assert(arr[3] == 5);
        assert(arr.length == 4);

        arr = [1];
        remove(0);
        // []
        assert(arr.length == 0);
    }
}
```

2. **Remove array element by copying last element into to the place to remove**

* Deleting an element creates a gap in the array.&#x20;
* One trick to keep the array compact is to move the last element into the place to delete.

```solidity
contract ArrayReplaceFromEnd { 

uint[] public arr;

    function remove(uint index) public {
        // Move the last element into the place to delete
        arr[index] = arr[arr.length - 1];
        // Remove the last element
        arr.pop();
    }

    function test() public {
        arr = [1, 2, 3, 4];
    
        remove(1);
        // [1, 4, 3]
        assert(arr.length == 3);
        assert(arr[0] == 1);
        assert(arr[1] == 4);
        assert(arr[2] == 3);
    
        remove(2);
        // [1, 4]
        assert(arr.length == 2);
        assert(arr[0] == 1);
        assert(arr[1] == 4);
    }
}
```





{% hint style="danger" %}
Be careful with Arrays because of gas costs due to iteration over elements! \
Might be better off using mappings.
{% endhint %}

## Memory Arrays

**Only fixed size memory array.** Dynamic array cannot be created in memory.

```solidity
contract Demo {

  function initialize() public pure returns(uint) {
        //fixed size of 2
        uint[] memory val = new uint[](2);
        
        val[0] = 100;        
        val[1] = 99;
        return (val[0] + val[1]);       
    }
}
```



### Arrays of structs

* [https://ethereum.stackexchange.com/questions/115575/dynamic-in-memory-array-declaration-and-assignment-in-0-8-10](https://ethereum.stackexchange.com/questions/115575/dynamic-in-memory-array-declaration-and-assignment-in-0-8-10)
* [https://ethereum.stackexchange.com/questions/87903/how-to-declare-an-array-of-structs-as-storage-variables](https://ethereum.stackexchange.com/questions/87903/how-to-declare-an-array-of-structs-as-storage-variables)

If you want to initialize dynamically-sized arrays, you have to assign the individual elements:

```csharp
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract C {
    function f() public pure {
        uint[] memory x = new uint[](3);
        x[0] = 1;
        x[1] = 3;
        x[2] = 4;
    }
}
```



## Layout in storage

### Dynamic arrays

* Storage slot of a dynamic array stores the length of the array; and updates it accordingly.
* Dynamic array elements are stored elsewhere.&#x20;
* Why: compiler has no way of knowing how many elements there would be; cos dynamic, so it cannot store the elements consecutively starting from the storage slot at which the dynamic array is declared. Danger of clashing into the next storage slot which could hold some other state variable.&#x20;

#### Location of array elements

hash(slot) = location of the first array element, and the rest follow

```solidity
// hash(slot = dynamicArraySlot)
uint256(keccak256(abi.encode(dynamicArraySlot)));
```

### Nested dynamic arrays

```solidity
uint256[][]s;    //storage location 0

s.push();       // push an empty dynArr, s[0]
s[0].push(4);
s[0].push();    //empty uint = 0
s[0].push(6);    
```

<figure><img src="../../../.gitbook/assets/image (352).png" alt=""><figcaption></figcaption></figure>

* s points to another array, s\[0]
* s\[0] points to the series of elements, s\[0] = \[4,0,6]

<figure><img src="../../../.gitbook/assets/image (357).png" alt=""><figcaption><p>example: if s[1] =5 and s[2] = 1</p></figcaption></figure>

* s\[1] pointer would be stored beside s\[0], and so forth

{% hint style="info" %}
storage layout  of dynamic nested arrays: [https://www.youtube.com/watch?v=Zi4BANKFNP8](https://www.youtube.com/watch?v=Zi4BANKFNP8)
{% endhint %}

## Storage pointers&#x20;

* myArray is not a separate unique array that is created from coders.&#x20;
* its a pointer to the location in storage where coders resides.
* modifying values through the pointer will affect both myArray and coders.

```solidity
contract fellowCoders {
  
    // Initialising array coders
    uint[] public coders;

    function pushCoders() public {
        coders.push(1);
        coders.push(2);
    }
    
    function fuckAround() public returns(uint256){

        //Creates storage pointer to coders. DOES NOT COPY ARRAY
        uint[] storage myArray = coders;
        
        // this creates a copy in memory
        uint[] memory memArray = coders;

        // Overwrites coders[0]. coders[0] 1 -> 0
        myArray[0] = 0;

        // returns 1 
        return memArray[0]; 
    } 
}
```

### Dangling references to storage array elements

1. add(): Create a dynamic array with a single nested dynamic array **uint256\[]\[] s**, such that **s\[0] = \[4,0,6]**
2. fuckAround(): create a storage pointer to the last element in s, which is a pointer to s\[0]; since there is only 1 element.
3. pop s\[0]
4. push some hex value into the same storage location&#x20;

<figure><img src="../../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>

```solidity
contract Storage {
    uint256[][] s;

    function add() public {
        s.push();
        s[0].push(4);
        s[0].push();
        s[0].push(6);
    }

    function pop() public {
        s.pop();
    }

    function fuckAround() public {
        // ptr := s[0], reference to the storage location
        uint[] storage ptr = s[s.length - 1];
        // remove s[0]
        s.pop();
        // push a value into the str location of s[0][0]
        ptr.push(0x42);
        // re-assign s[0]. references same location
        s.push();
        // s[0][0] == 0x42, cos of clashing storage location
        assert(s[s.length - 1][0] == 0x42);
    }

    // HELPER TO READ FROM STORAGE SLOTS
    function readStorageSlot(uint256 i) public view returns (bytes32 content) {
        assembly {
            content := sload(i)
        }
    }

    function getLocationOfDynamicArray(uint dynamicArraySlot) public pure returns (uint) {
        // dynamicArraySlot: the slot that the dynamic array itself sits in
        return uint256(keccak256(abi.encode(dynamicArraySlot)));
    }
}
```

<figure><img src="../../../.gitbook/assets/image (359).png" alt=""><figcaption></figcaption></figure>
