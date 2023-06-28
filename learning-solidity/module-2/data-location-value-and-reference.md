# Data Location: Value & Reference

Solidity variables can be classified according to their **data location**. There are two types:

1. Value types&#x20;
2. Reference types

### Value type

A value type stores its data directly in the memory it owns. Variables of this type are **duplicated** whenever they appear in functions or assignments. A value type will maintain an **independent copy** of any duplicated variables.&#x20;

* `uint`, `int`, `address`, `bool`, `enum`, `bytes`

{% hint style="info" %}
Therefore, a change in the value of a duplicated variable will not affect the original variable.
{% endhint %}

### Reference type

A reference type stores the address of the data’s location; acts as a pointer to a value stored elsewhere.&#x20;

If you examine the location of where the reference type was created, it will contain the pointer directing us to the actual location of the value; not the value itself.

* `mapping`, `struct`, `array`
  * string arrays, byte arrays, array members, fixed/dynamic arrays

{% hint style="info" %}
* When a reference type variable is assigned to another variable or when a reference type variable is sent as an argument to a function, EVM creates a new variable instance and copies the pointer from the original variable into the target variable.&#x20;
* This is known as passing by reference.&#x20;
* Both the variables are pointing to the same address location. Both the variables will share the same values and change committed by one is reflected in the other variable.
{% endhint %}
