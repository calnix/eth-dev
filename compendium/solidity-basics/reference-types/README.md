---
description: mapping, struct, arrays
---

# Reference Types

Reference types, unlike value types, don’t store their values directly within the variables themselves.&#x20;

They store the address of the situation where the worth is stored. The variable holds the pointer to a different memory location that holds the particular data.

### **Passing by reference**

* Both the variables are pointing to the same address location. Both the variables will share the same values and change committed by one is reflected in the other variable.
* When a reference type variable is assigned to another variable or when a reference type variable is sent as an argument to a function, EVM creates a new variable instance and copies the pointer from the original variable into the target variable.&#x20;

\
