# Traces

Forge can produce traces either for **failing tests** (`-vvv`) or all tests (`-vvvv`).

Traces follow the same general format:

```solidity
  [<Gas Usage>] <Contract>::<Function>(<Parameters>)
    ├─ [<Gas Usage>] <Contract>::<Function>(<Parameters>)
    │   └─ ← <Return Value>
    └─ ← <Return Value>
```

If your terminal supports colour, the traces will also come with a variety of colours:

* **Green**: For calls that do not revert
* **Red**: For reverting calls
* **Blue**: For calls to cheat codes
* **Cyan**: For emitted logs
* **Yellow**: For contract deployments

The gas usage (marked in square brackets) is for the entirety of the function call.

```solidity
  [24661] OwnerUpOnlyTest::testIncrementAsOwner()
    ├─ [2262] OwnerUpOnly::count()
    │   └─ ← 0
    ├─ [20398] OwnerUpOnly::increment()
    │   └─ ← ()
    ├─ [262] OwnerUpOnly::count()
    │   └─ ← 1
    └─ ← ()
```

{% hint style="info" %}
Sometimes the gas usage of one trace does not exactly match the gas usage of all its subtraces.

The gas unaccounted for is due to some extra operations happening between calls, such as arithmetic and store reads/writes
{% endhint %}
