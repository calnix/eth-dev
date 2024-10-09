# adv\_deploy() + Testing

* OpenSea testnet is on Rinkeby

{% tabs %}
{% tab title="adv_deploy.py" %}
```python
from brownie import AdvancedCollectible
from scripts.helpful_scripts import get_account, OpenSeaURL, get_contract, fund_with_link
from brownie import config, network 

# OpenSea Testnet is on rinkeby
def deploy():
    account = get_account()
    advanced_collectible = AdvancedCollectible.deploy(
        get_contract("vrf_coordinator").address,
        get_contract("link_token").address,
        config["networks"][network.show_active()]["fee"],
        config["networks"][network.show_active()]["keyhash"], {"from": account}, publish_source = config["networks"][network.show_active()].get("verify", False) )

    # fund w/ link
    fund_with_link(advanced_collectible.address)

    # create
    create_tx = advanced_collectible.createCollectible({"from": account})
    create_tx.wait(1)
    print("New NFT has been created")
    return advanced_collectible, create_tx

def main():
    deploy()
```
{% endtab %}
{% endtabs %}

We can deploy to either our local environment or rinkeby for testing. Unit testing will be done locally, while integration testing will be conducted on rinkeby.

### Unit Testing

{% code title="test_advanced_collectible.py" %}
```python
from brownie import AdvancedCollectible, network
from scripts.helpful_scripts import get_account, LOCAL_BLOCKCHAIN_ENV, FORKED_LOCAL_ENV, get_contract
from scripts.AdvCollectible.adv_deploy import deploy
import pytest


def test_can_create_advanced_collectible():
    # Arrange
    if network.show_active() not in LOCAL_BLOCKCHAIN_ENV:
        pytest.skip()
    
    # Act
    advanced_collectible, create_tx = deploy()
    requestId = create_tx.events["requestCollectible"]["requestId"]
    random_number = 777

    ## function callBackWithRandomness(bytes32 requestId,uint256 randomness,address consumerContract)
    get_contract("vrf_coordinator").callBackWithRandomness(requestId, random_number, advanced_collectible.address, {"from": get_account()})

    # Assert
    assert advanced_collectible.tokenCounter() == 1 
    assert advanced_collectible.tokenIdToBreed(0) == random_number % 3
```
{% endcode %}

* Since this will be executed on development network(ganache-cli), we will need mocks deployed.
* Additionally, we will need to simulate the off-chain response to the VRF coordinator, and it calling callBackWithRandomness.
* Here we opt to pass 777 as our random number.
* Assert that NFT is minted
  * tokenCounter incremented
  * breed assigned

### Integration Testing

{% code title="test_advanced_integration.py" %}
```python
from brownie import AdvancedCollectible, network
from scripts.helpful_scripts import get_account, LOCAL_BLOCKCHAIN_ENV, FORKED_LOCAL_ENV, get_contract
from scripts.AdvCollectible.adv_deploy import deploy
import pytest
import time


def test_can_create_advanced_collectible_integration():
    # Arrange
    if network.show_active() in LOCAL_BLOCKCHAIN_ENV:
        pytest.skip("Only for integration testing on rinkeby")
    
    # Act
    advanced_collectible, create_tx = deploy()
    time.sleep(60)

    # Assert
    assert advanced_collectible.tokenCounter() == 1 
```
{% endcode %}

This will be executed on rinkeby.

{% hint style="warning" %}
Even if we are testing only one function (-k), brownie will collect and compile all tests - so if there are errors in the others, they will be surfaced.
{% endhint %}

### Interacting with rinkeby deployment

```python
from brownie import AdvancedCollectible
from scripts.helpful_scripts import fund_with_link, get_account, get_contract
from web3 import Web3

# for interacting with live rinkeby deployed contract
def main():
    account = get_account()
    advanced_collectible = AdvancedCollectible[-1]
    
    # fund with link
    fund_with_link(advanced_collectible.address, amount=Web3.toWei(0.1, "ether"))

    # create
    create_tx = advanced_collectible.createCollectible({"from": account})
    create_tx.wait(1)
    print("New NFT has been created")
    return advanced_collectible, create_tx
```

This will reference our most recent rinkeby deployment, as per our build folder. If there was no previous deployment or build folder was deleted, `AdvancedCollectible[-1]` will error out.
