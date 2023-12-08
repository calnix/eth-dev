# Value Types

A value type in computer programming is a coded object that involves **memory allocation directly where it is created**. Value types are commonly contrasted to reference types that instead act as pointers to a value that is stored elsewhere.

These types of variables are always **passed by value**. The variables are copied wherever they are used in function arguments or assignment.

You do not need to specify the location when passing it as a parameter into a function.

* (u)int
* address
* boolean

{% hint style="danger" %}
Both string and bytes are arrays. String arrays have no length or index-access. So you cannot access a specific element in it.

Bytes for arbitrary length raw data.

From a reference perspective, we are going to treat string as a reference type. &#x20;
{% endhint %}

![](<../../../.gitbook/assets/image (293).png>)

### Integers: (u)int

`uint` is actually an alias for `uint256`, a 256-bit unsigned integer. You can declare `uints`with less bits — `uint8`, `uint16`, `uint32`, etc. But in general you want to simply use `uint` except in specific cases.

### Signed integers

* `Int8` —>   \[-128 : 127]
* `Int16` —> \[-32768 : 32767]
* `Int32` —> \[-2147483648 : 2147483647]

### **Unsigned integers**

* `UInt8` — \[0 : 255]
* `UInt16` — \[0 : 65535]
* `UInt32` — \[0 : 4294967295]
* `UInt64` — \[0 : 18446744073709551615]

{% hint style="info" %}
Normally there's no benefit to using these sub-types because Solidity reserves 256 bits of storage regardless of the `uint` size. For example, using `uint8` instead of `uint` (`uint256`) won't save you any gas.
{% endhint %}

{% hint style="warning" %}
But there's an exception to this: within `struct`
{% endhint %}

If you have multiple `uint` inside a struct, using a smaller-sized `uint` when possible will allow Solidity to pack these variables together to take up less storage. For example:

```fsharp
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` will cost less gas than `normal` because of struct packing
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```

**For this reason, inside a struct you'll want to use the smallest integer sub-types you can get away with.**

&#x20;You'll also want to cluster identical data types together (i.e. put them next to each other in the struct) so that Solidity can minimize the required storage space. For example, a struct with fields `uint c; uint32 a; uint32 b;` will cost less gas than a struct with fields `uint32 a; uint c; uint32 b;` because the `uint32` fields are clustered together.

###
