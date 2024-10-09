# Brownie console

is an interactive shell. meant for ad-hoc testing and experimenting.

> brownie console

console will be launched on a brand new local ganache chain, as a test environment. &#x20;

* brand new environment -> deploy contract first
  * account = accounts\[0]
  * simple\_storage = SimpleStorage.deploy({"from": account})
* len(SimpleStorage) = 1  -> one instance of contract deployed

can run script line by line and check results.
