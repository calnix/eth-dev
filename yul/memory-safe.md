# memory-safe

By default solidity code is considered memory safe, but the `inline assembly` blocks need to be declared explictly using `assembly ("memory-safe") { ... }`.&#x20;

The compiler does **NOT** check that it actually is memory safe but turns on the above functionality believing it is actually memory safe on the programmer’s promise. Declaring it as memory safe but not actually being memory safe leads to **undefined behaviours** .

**What is a memory safe assembly block?**&#x20;

TLDR: assembly that respect solidity memory layout. It’s memory safe when the following memory blocks are accessed :

1. Memory allocated by yourself respecting solidity layout i.e **reading from free memory pointer** `0x40` and **updating (incrementing) it correctly after any allocations.**
2. Memory **allocated by Solidity**, e.g. memory within the bounds of a memory array you reference.
3. **The scratch space** between memory offset 0 and 64 mentioned above.
4. Temporary memory that is located _after_ the value of the free memory pointer at the beginning of the assembly block, i.e. memory that is “allocated” at the free memory pointer without updating the free memory pointer.
5. The assmebly block that doesn’t have any consecutive allocations. For example, if the assembly block is the last piece of code inside the function, it is safe by default as all memory is wiped after function call finishes.

Link: [https://0xpranay.github.io/solidity-notes/Internals.html](https://0xpranay.github.io/solidity-notes/Internals.html)

{% hint style="info" %}
Using memory-safe assembly does not enable any safety checks or restrictions. Quite the opposite.&#x20;

The compiler cannot determine on its own whether the code is memory-safe or not. By using this feature, you’re declaring that it is and signaling that the compiler can do things that would not be safe to do otherwise. If it turns out it’s not memory-safe, bad things can happen.
{% endhint %}
