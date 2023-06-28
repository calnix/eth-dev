# multiple address(this)

#### Would it be more efficient to store the contract address as a variable during the constructor rather than using address(this) in multiple functions?

* Yes, it would.
* address(this) uses the `ADDRESS` opcode each time. cost 1 gas to push to the stack each time.
