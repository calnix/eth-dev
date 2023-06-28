# assume vs bound

* Supposed to use bound for most things,
* Only use vm.assume for very narrow checks

For broad checks, such as ensuring a `uint256` falls within a certain range, you can bound your input with the modulo operator or Forge Standard's [`bound`](https://book.getfoundry.sh/reference/forge-std/bound.html) method.

\
[https://book.getfoundry.sh/cheatcodes/assume.html](https://book.getfoundry.sh/cheatcodes/assume.html)
