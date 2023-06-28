---
description: >-
  https://blog.chain.link/events-and-logging-in-solidity/  |
  https://www.youtube.com/watch?v=w18c9HLEuBs
---

# Logging, Events, Solidity, Bloom Filter



![](<../.gitbook/assets/image (324).png>)

* **Address**: address of contract that emitted the event
* **Topics**: indexed parameters of events
* **Data**: abi-encoded non-indexed parameters of events
  * have to decode using the abi of the contract
  * if contract is verified on etherscan, can view in decoded mode 'dec'

```python
from scripts.helpful_scripts import get_account
from brownie import SimpleStorage, config, network

def deploy():
    account = get_account()
    simple_storage = SimpleStorage.deploy({"from": account}, publish_source=config["networks"][network.show_active()].get("publish_source", False),)

    tx = simple_storage.store(1, {"from": account})
    tx.wait(1)

    print(tx.events)
    print(tx.events[0]["oldNumber"])
    print(tx.events[0]["newNumber"])
    print(tx.events[0]["addedNumber"])
    print(tx.events[0]["sender"])


def main():
    deploy()
```

* [https://www.youtube.com/watch?v=w18c9HLEuBs](https://www.youtube.com/watch?v=w18c9HLEuBs)
* [https://github.com/PatrickAlphaC/brownie-events-logs/blob/main/scripts/deploy\_and\_store.py](https://github.com/PatrickAlphaC/brownie-events-logs/blob/main/scripts/deploy\_and\_store.py)
