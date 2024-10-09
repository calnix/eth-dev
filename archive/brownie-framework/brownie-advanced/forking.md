# Forking

Creating a copy of the mainnet to be used within our internal environment

* use dummy acc
* but do not deploy mocks
* add env variable: FORKED\_BLOCKCHAIN\_ENV&#x20;

### Create mainnet-fork-dev, add to brownie networks

1. add fork network to brownie \[call it mainnet-fork-dev]
2. add it to development category
3. brownie networks add **development** mainnet-fork-dev cmd=ganache-cli host=http:// 127.0.0.1 fork=**'https://mainnet.infura.io/v3/$WEB3\_INFURA\_PROJECT\_ID'** accounts=10 mnemonic=brownie port=8545

* the single quotes in the fork link -> run as is&#x20;
* without single quotes: our env variable would be actualized into the URL

Infura forking typically has issues, so alchemy is suggested:

brownie networks add development mainnet-fork-dev cmd=ganache-cli host=http://127.0.0.1 fork=https://eth-mainnet.alchemyapi.io/v2/j13raIIaPyTnV9ZVv2UnQm48PUFcPDkL accounts=10 mnemonic=brownie port=8545

{% hint style="danger" %}
\*\* mainnet-fork-dev is a development env. it wont have anything in deployment. cannot interact with existing contracts.
{% endhint %}

{% hint style="warning" %}
Note: brownie comes with `mainnet-fork`, no idea what the setup is. PatrickACollins: he deletes and use mainnet-fork-dev as default.
{% endhint %}

### Run testing

brownie test --network mainnet-fork-dev

```python
import pytest
from brownie import FundMe, accounts, network, exceptions
from multiprocessing.connection import wait
from scripts.fund_and_withdraw import fund 
from scripts.helpful_scripts import *
from scripts.deploy import deploy_fund_me

def test_fund_withdraw(): 
    account = get_account() 
    fund_me = deploy_fund_me() 
    print(f"THIS WORKS {fund_me.address}") 
    
    entrance_fee = fund_me.getEntranceFee() + 100 
    tx = fund_me.fund({"from": account, "value": entrance_fee}) 
    tx.wait(1) print(entrance_fee) 
    assert fund_me.addressToAmountFunded(account.address) == entrance_fee 
    
    tx2 = fund_me.withdraw({"from": account}) 
    tx2.wait(1) 
    assert fund_me.addressToAmountFunded(account.address) == 0
```

* use our own dummy accounts
* deploy new instances of our contracts (fork is dev).&#x20;
* ensure ganacheUI isn't running, else we cannot reference the cloned mainnet contracts

{% hint style="info" %}
can take awhile for the forked node to respond. cos its coming from alchemy API. wait till you get response.

\-- also, if you have ganache running, close it.&#x20;

\-- for some reason brownie attaches to it instead of pulling data from the fork host API.&#x20;

\-- test then fails, cos' there are no deployed contracts to pull price.
{% endhint %}

### Git stuff

git init -b main (init branch called main) git config user.name "" git config user.email "calnix.289@gmail.com"

::push :: git add . \_> cache files to staging area git status -> shows files in staging to be pushed

git commit -m 'first commit' git remote add origin git push -u origin main
