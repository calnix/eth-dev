# array

**Can be fixed or dynamic sized**

* T\[k] -> fixed size of type T, with k elements
* T\[] -> dynamic size of type T
* T\[]\[5] -> 5 dynamic-sized arrays of T (read in reversed)

**Arrays have two methods:**

* length
* push(element)

{% hint style="danger" %}
Be careful with Arrays because of gas costs due to iteration over elements! \
Might be better off using mappings.
{% endhint %}

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
