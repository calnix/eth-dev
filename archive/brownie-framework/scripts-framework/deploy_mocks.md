# deploy\_mocks()

When deploying to local blockchain environments that are not forks (ganache-cli, ganacheUI), we need to deploy mock contracts of external contracts we import .

When deploying to Forked environments, no need for mocks. We will refer to existing deployments via their addresses on the respective chain we are forking.

## Iteration 1&#x20;

{% tabs %}
{% tab title="deploy.py" %}
```python
from brownie import FundMe
from brownie import network, config
from scripts.helpful_scripts import *
from scripts.helpful_scripts import LOCAL_BLOCKCHAIN_ENV

def deploy_fund_me():
    account = get_account()
    print(f"Deploying on ....{network.show_active()}")
    
    if network.show_active() not in LOCAL_BLOCKCHAIN_ENV:
        price_feed_address = config["networks"][network.show_active()]["eth_usd_price_feed"]

    else: #deploy mock of price feed on internal chain: development
        deploy_mocks()
        price_feed_address = MockV3Aggregator[-1].address 
```

Here the external contract we need is the chainlink ETH/USD price feed contract to pass into AggregatorV3Interface.
{% endtab %}

{% tab title="helpful_scripts.py" %}
```python
from brownie import MockV3Aggregator

DECIMAL_PLACES = 8 #to resemble eth/usd price feed on mainnet aggregator
STARTING_PRICE = (2000*10**8)

def deploy_mocks():
        print(f"The active network is {network.show_active()}")
        print("Deploying.....")
        if len(MockV3Aggregator) <= 0:
            MockV3Aggregator.deploy(DECIMAL_PLACES,STARTING_PRICE,{"from":get_account()})  #contract object 
        print(f"Mocks Deployed at: {MockV3Aggregator[-1].address}")
```
{% endtab %}

{% tab title="brownie-config.yaml" %}
```yaml
networks:
  default: development  #default is development, un;ess we specify here.
  rinkeby: 
    eth_usd_price_feed: '0x8A753747A1Fa494EC906cE90E9f37563A8AF630e'
    verify: True
  mainnet-fork-dev: 
    eth_usd_price_feed: '0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419'  #mainnet address (for forking reference) - https://docs.chain.link/docs/ethereum-addresses/
    verify: False
  development:
    verify: False
  ganache-local:
    verify: False
    
```
{% endtab %}
{% endtabs %}

#### If deploying to live chain (mainnet, rinkeby), price\_feed\_address will reference the chainlink ethusd contract address on the corresponding deployed.

We store chainlink's pricefeed contract's corresponding address for each live chain in brownie-config.yaml, under the variable: eth\_usd\_price\_feed.

If we are deploying to rinkeby, price\_feed\_address == eth\_usd\_price\_feed == 0x8A753747A1Fa494EC906cE90E9f37563A8AF630e

#### If deploying to internal chain, like development

deploymocks_()_ is called -> stored in helpful\_scripts.py

* checks if a prior instance exists
* if there are not prior deployments, it will deploy a fresh mock contract.
* if a prior deployment exists, nothing is done within deploy\_mocks(),
  * execution goes back deploy.py line15: \
    price\_feed\_address = MockV3Aggregator\[-1].address&#x20;

## Iteration 2: get\_contract() + deploy\_mocks()

{% tabs %}
{% tab title="deploy.py" %}
```python
from brownie import Lottery
from scripts.helpful_scripts import get_account, get_contract()

def deploy():
    account = get_account()
    Lottery.deploy(
        get_contract("ethusd_pricefeed").address, 
        get_contract("vrf_coordinator").address,
        get_contract("link_token").address,
        config["networks"][network.show_active()]["fee"],
        config["networks"][network.show_active()]["keyhash"],{"from":account}, 
        publish_source = config["networks"][network.show_active()].get("verify", False))
        # get verify key, if it doesnt exist - default to false.
    print(".....Deployed lottery!.....")
    
def main():
    deploy()
```
{% endtab %}

{% tab title="helpful_scripts.py" %}
```python
from brownie import network, accounts,config, 
from brownie import interface
from brownie import MockV3Aggregator,VRFCoordinatorMock,LinkToken, Contract

LOCAL_BLOCKCHAIN_ENV = ["development", "ganache-local"]
FORKED_LOCAL_ENV = ["mainnet-fork", "mainnet-fork-dev"]

#dictionary
#our reference_name: Contract Name as per import
contract_to_mock = {
    "ethusd_pricefeed": MockV3Aggregator,
    "link_token": LinkToken,
    "vrf_coordinator": VRFCoordinatorMock
    }

def get_contract(contract_name):
    """
    function will grab the mainnet/testnet contract addresses from the brownie config if defined.
    Otherwise, it will deploy a mock version of that contract, and return that mock contract address.
        Args:
            contract_name (string)
        Return:
            brownie.network.contract.ProjectContract: Most recently deployed version of this contract
    """
    
    contract_type = contract_to_mock[contract_name]

    if network.show_active() in LOCAL_BLOCKCHAIN_ENV:  #forks wont need mocks
        if len(contract_type) <0:           # check if got prior deployment
            deploy_mocks()
        contract = contract_type[-1]  #grab most recent
    else: 
        # get address from config
        # need abi of deployed contract so we can interact. 
        contract_address = config["networks"][network.show_active()][contract_name]
        contract = Contract.from_abi(contract_type.name, contract_address, contract_type.abi)
        # creating Contract object from abi
        # contract_type.abi pulls abi from build/dependencies folder: AggregatorV3Interface.json
    return contract
    

DECIMALS = 8          #to resemble eth/usd price feed on mainnet aggregator
STARTING_PRICE = (2000*10**8)

def deploy_mocks(decimals = DECIMALS, start_price= STARTING_PRICE):
    account = get_account()
    print(f"The active network is {network.show_active()}")
    print("Deploying.....")
    MockV3Aggregator.deploy(decimals,start_price,{"from":account})  #contract object 
    link_token = LinkToken.deploy({"from":account})
    VRFCoordinatorMock.deploy(link_token.address,{"from":account})
    #print(f"Mocks Deployed at: {MockV3Aggregator[-1].address}")
    print(".....All Mocks Deployed!.....")
  
```
{% endtab %}

{% tab title="brownie-config.yaml" %}
```yaml
dependencies:
  - smartcontractkit/chainlink-brownie-contracts@1.1.1
  - OpenZeppelin/openzeppelin-contracts@3.4.0
  
compiler:
  solc:
    remappings:
      - '@chainlink=smartcontractkit/chainlink-brownie-contracts@1.1.1'
      - '@openzeppelin=OpenZeppelin/openzeppelin-contracts@3.4.0'

dotenv: .env

networks:
  default: development
  development:
    keyhash: '0x2ed0feb3e7fd2022120aa84fab1945545a9f2ffc9076fd6156fa96eaff4c1311'
    fee: 100000000000000000
  rinkeby:
    vrf_coordinator: '0xb3dCcb4Cf7a26f6cf6B120Cf5A73875B7BBc655B'
    eth_usd_price_feed: '0x8A753747A1Fa494EC906cE90E9f37563A8AF630e'
    link_token: '0x01BE23585060835E02B77ef475b0Cc51aA1e0709'
    keyhash: '0x2ed0feb3e7fd2022120aa84fab1945545a9f2ffc9076fd6156fa96eaff4c1311'
    fee: 100000000000000000
    verify: True
  mainnet-fork:
    eth_usd_price_feed: '0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419'
    verify: False

wallets:
  from_key: ${PRIVATE_KEY}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
deploy\_mocks() -> deploys mocks of each type every time its called within get\_contracts()

when calling get\_contacts(ethusd\_pricefeed), in deploy.py

&#x20;       if len(contract\_type) < 0:\
&#x20;           deploy\_mocks() contract = contract\_type\[-1]

ALL mocks are deployed. and again each time we call get\_contracts.
{% endhint %}

### Explanation

Create a dictionary `contract_to_mock`, which stores the actual contract names as values, and our reference names as keys.&#x20;

* "eth\_usd\_price\_feed": MockV3Aggregator
* contract\_type = MockV3Aggregator

getcontract takes a parameter, contract\_name, which is passed into the dictionary to obtain the true name of the contract.

#### if deploying to LOCAL\_BLOCKCHAIN\_ENV

mocks will be required. check if prior mock exists, if it does not, deploy\_mocks() is ran.

contract = contract\_type\[-1] -> contract = MockV3Aggregator\[-1] &#x20;

This creates a Contract container called contract which holds the mock contracts. To access their address: `contract.address`&#x20;

#### &#x20;Else: no mocks needed

Access deployed contracts via their livenet addresses and abi.

contract address is obtained from brownie-config.yaml\
&#x20;       contract\_address = config\["networks"]\[network.show\_active()]\[contract\_name]

Beyond the address, we will need the contract's ABI to know how to interact with it.&#x20;

Together, we will create a brownie Contract container object:

```python
contract = Contract.from_abi(contract_type.name, contract_address, contract_type.abi)
```

The ABI is accessed via contract\_type.abi -> MockV3Aggregator.abi

The ABI is generated and stored in `build/contracts/dependencies` as part of our import statement setup.

{% hint style="info" %}
Once we setup our import statements with the correct references and compile the project, brownie automatically downloads the necessary contracts as part of our dependencies and generates their ABI.
{% endhint %}

