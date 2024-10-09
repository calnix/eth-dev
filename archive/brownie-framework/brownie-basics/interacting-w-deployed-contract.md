# Interacting w/ deployed contract

We previously deployed a contract onto rinkeby.

In the deployments folder, deployed contracts are organized into different folders based on their chainID.

* Rinkeby -> chainID: 4 -> we will find our deployed contract's json file located here.

This serves as an interface for our python scripts to interact with the contract that resides on chain.

For example, we can read and write to the contract as follows:

{% code title="read_value.py" %}
```python
from brownie import SimpleStorage, accounts, config

#[-1] for most recent deployment, [0] for first deployment
def read_contract():
    simple_storage = SimpleStorage[-1]   

    #read value
    print(simple_storage.retrieve())

    #update value
    account = accounts.add(config["wallets"]["wallet1"])
    simple_storage.store(5, {"from":account})

    print(simple_storage.retrieve())

def main():
    read_contract()
```
{% endcode %}

We can run the above script with: `brownie run scripts/read_value.py --network rinkeby`

![](<../../../.gitbook/assets/image (215).png>)

* read current value of favNum -> returns 3  (call)
* update favNum to 5&#x20;
  * transaction hash provided
  * can look up on etherscan
* read current value of favNum -> returns 5 (call)

{% hint style="info" %}
this means you can knock out some python script to interact with your contract after running deploy.py
{% endhint %}
