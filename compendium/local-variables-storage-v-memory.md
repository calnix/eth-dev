# Local Variables (Storage v Memory)

Only complex data types (arrays and structs) **default** to **storage inside functions**, while all others default to memory.

{% hint style="info" %}
Storage is a key/value store where keys and values are both 32 bytes. Memory is a byte-array. Memory starts off zero-size, but can be expanded in 32-byte chunks by simply accessing or storing memory at indices greater than its current size.\
[\
https://ethereum.stackexchange.com/questions/1232/difference-between-memory-and-storage](https://ethereum.stackexchange.com/questions/1232/difference-between-memory-and-storage)
{% endhint %}

A consequence of this design difference is that **storage is dynamic** and memory is not.&#x20;

Because arrays and structs are complex and could be of variable length, they are defaulted to storage, which has this key:value behaviour.&#x20;

Simpler variables like bool, uint, etc are not variable in length, and are therefore defaulted to memory, which is [cheaper than storage](https://ethereum.stackexchange.com/questions/23720/usage-of-memory-storage-and-stack-areas-in-evm/23722). So, think of the design choice as a compromise between flexibility and cost.

It is possible to [create memory arrays](https://solidity.readthedocs.io/en/develop/types.html#arrays), but once created they cannot be resized (check out the Allocating Memory Arrays section).

{% hint style="info" %}
For value types (booleans, integers, addresses ...) it's memory. For complex types (arrays, structs, maps) the default location depends on the context and can be [overridden](http://solidity.readthedocs.io/en/develop/types.html#data-location) by `memory` and `storage` keywords.
{% endhint %}

## [Do intermediate memory variables cost gas?](https://ethereum.stackexchange.com/questions/60042/do-intermediate-memory-variables-cost-gas)

```solidity
pragma solidity ^0.4.24;
contract A {
    uint t;

    function run() public returns(uint){
        uint startTime = 9;
        uint allowedTime = 7;

        uint v = startTime + allowedTime; // second test without v
        t = now + v;
        return t;
    }
}
```

#### Results

```
with v
  deploy 95237 gas
  func 41470 gas

without v
  deploy 95237 gas
  func 41470 gas
```

* Compiling with the optimizer leads to the same bytecode and gas cost.
* If the optimizer is not used, the compiler would produce more bytecode, and the gas cost would be higher.

#### Links&#x20;

* [https://medium.com/coinmonks/solidity-bits-storage-vs-memory-a54a650ea4ff](https://medium.com/coinmonks/solidity-bits-storage-vs-memory-a54a650ea4ff)
* [https://solidity-by-example.org/data-locations/](https://solidity-by-example.org/data-locations/)
