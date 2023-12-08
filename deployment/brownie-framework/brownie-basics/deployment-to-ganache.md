---
description: brownie automatically handles ganache initiation through ganache-cli
---

# Deployment to ganache

To proceed with deployment of the Smart Contract, we first need to import it into our script so we can interact with it.

```python
#import contract into script: 
#from brownie import .. , .., <name of contract>
from brownie import accounts, config, SimpleStorage

#put all the deplyment logic in one fucntion:
def deploy_simple_storage():
    #local ganache accounts
    account = accounts[0]

    #select which account to deploy from
    simple_storage = SimpleStorage.deploy({"from": account})
    print(simple_storage)
```

Specify which account to deploy from. Brownie will build,sign and send the transaction.

simple\_storage will store the return of the .deploy function -> it is a contract object.&#x20;

The output, when printing simple\_storage is the smart contract address:

![](<../../../.gitbook/assets/image (256).png>)

{% hint style="info" %}
When brownie compiles the project, it creates `ContractContainer` objects for each deployable contract.&#x20;

* This object is a container, used to access individual deployments.
* It is also used to deploy new contracts.&#x20;
* For example, SimpleStorage is a ContractContainer object which we import from brownie
* [`ContractContainer.deploy`](https://eth-brownie.readthedocs.io/en/stable/api-network.html#ContractContainer.deploy) is used to deploy a new contract.
* Calling [`ContractContainer.deploy`](https://eth-brownie.readthedocs.io/en/stable/api-network.html#ContractContainer.deploy) returns a [`ProjectContract`](https://eth-brownie.readthedocs.io/en/stable/api-network.html#brownie.network.contract.ProjectContract) object. The returned object is also appended to the [`ContractContainer`](https://eth-brownie.readthedocs.io/en/stable/api-network.html#brownie.network.contract.ContractContainer)
* hence, len(ContractContainer) -> instances of deployment
{% endhint %}

## Interacting with your Contracts

Once a contract has been deployed, you can interact with it via via _calls_ and _transactions_.

> * **Transactions** are broadcast to the network and recorded on the blockchain. They cost ether to run, and are able to alter the state to the blockchain.
> * **Calls** are used to execute code on the network without broadcasting a transaction. They are free to run, and cannot alter the state of the blockchain in any way. Calls are typically used to retrieve a storage value from a contract using a getter method.

## Note:

* deploy.py:&#x20;
* brownie does not automatically deploy all the contracts in our contracts folder.&#x20;
* we must specify what we want deployed through our python code.
