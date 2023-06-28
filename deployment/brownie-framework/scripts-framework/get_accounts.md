# get\_accounts

## Account types

1. &#x20;accounts\[0]  -- dummy ganache accounts
2. accounts.**add**("env")  -- private key from env file&#x20;
   1. \->  accounts.**add**(config\["wallets"]\["wallet1"])
3. accounts.**load**("id") -- load from brownie accounts list&#x20;

## Iteration 1

{% tabs %}
{% tab title="deploy.py" %}
```python
from brownie import FundMe
from brownie import network, config
from scripts.helpful_scripts import *
from scripts.helpful_scripts import LOCAL_BLOCKCHAIN_ENV

def deploy_fund_me():
    account = get_account()
    print(f"Deploying on .....{network.show_active()}")
    
    if network.show_active() not in LOCAL_BLOCKCHAIN_ENV:
        price_feed_address = config["networks"][network.show_active()]["eth_usd_price_feed"]

    else: #deploy mock of price feed on internal chain: development
        deploy_mocks()
        price_feed_address = MockV3Aggregator[-1].address 

    fund_me = FundMe.deploy(price_feed_address,
    {"from": account}, 
    publish_source=config["networks"][network.show_active()].get("verify"))

    print(f"Contract deployed to: {fund_me.address}")
    return fund_me

def main():
    deploy_fund_me() 
    #Python will automatically run main() on execution.

```
{% endtab %}

{% tab title="helpful_scripts.py" %}
```python
from brownie import accounts, network, config
from brownie import MockV3Aggregator

LOCAL_BLOCKCHAIN_ENV = ["development", "ganache-local"]
FORKED_LOCAL_ENV = ["mainnet-fork", "mainnet-fork-dev"]

def get_account(): 
 if network.show_active() in LOCAL_BLOCKCHAIN_ENV 
 or network.show_active() in FORKED_LOCAL_ENV: 
   return accounts[0] #use ganache generated account. 
 else: 
   return accounts.add(config["wallets"]["wallet1"]) 
   #look in config.yaml
 
```
{% endtab %}

{% tab title="brownie-config.yaml" %}
```yaml
networks:
  default: development  #default is development, unless we specify here.
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
    
wallets:
  wallet1: ${PRIVATE_KEY}

```
{% endtab %}

{% tab title=".env" %}
```bash
export PRIVATE_KEY = 0x57979d527794c1adbe2dc140d0aad4f1adada194aba46b5175a395fe71887c25
export WEB3_INFURA_PROJECT_ID = cd3a18407dfe4d2597f756dafe2804d6
export ETHERSCAN_TOKEN = JVPJ4F7ED7AHE4UQ9E9MUXP665SS93HHU7
```
{% endtab %}
{% endtabs %}

### **Account**

`account = get_account()`

* get\_account is defined in helpful\_scripts
* Function will check which network we are deploying to. If it is in LOCAL\_BLOCKCHAIN\_ENV / FORKED\_LOCAL\_ENV, a dummy ganache account is used.

{% hint style="info" %}
forking mainnet into a private env, so can use dummy accounts.
{% endhint %}

* Else, (i.e. live chains: mainnet, testnet), MetaMask wallet is used.
  * pulls private key from wallet1 defined in brownie-config.yaml&#x20;
  * which in turn references the PK stored in the .env file
  * we do this, as opposed to hardcoding into the yaml file, so that on a git push, the .env is not uploaded to github, thereby exposing our private key.

In this version of get\_accounts(), we only utilized dummy accounts and adding accounts from Private Key. We shall see the third method in the next interation.

## Iteration 2

{% tabs %}
{% tab title="First Tab" %}
```python
from brownie import network, accounts,config, Contract
from brownie import MockV3Aggregator

DECIMAL_PLACES = 8          #to resemble eth/usd price feed on mainnet aggregator
STARTING_PRICE = (2000*10**8)

LOCAL_BLOCKCHAIN_ENV = ["development", "ganache-local"]
FORKED_LOCAL_ENV = ["mainnet-fork", "mainnet-fork-dev"]

def get_account(index=None,id=None):
    if index:
        #if index was passed return ganache account
        return accounts[index]
    
    if id:
        return accounts.load(id)

    if network.show_active() in LOCAL_BLOCKCHAIN_ENV or network.show_active() in FORKED_LOCAL_ENV:
        return accounts[0]          #use ganache generated account.  
      
    else: #look in config.yaml
        return accounts.add(config["wallets"]["wallet1"])  
```
{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

### def get\_account(index=None,id=None):

#### both parameters, `index` and `id` are optional, as their default value is set to `None`.

* if index parameter is passed, function will return a dummy ganache account based on the parameter
  * account\[index]
* if id is passed, for example, get\_account(id=freecodecamp)
  * function will reference stored accounts in brownie of the id name
  * can store an account by: brownie accounts new \<id>&#x20;
    * pass the private key, add 0x before it.
    * brownie will request for a password to hash it.

**If neither index nor id are passed,**&#x20;

* and deployment into LOCAL\_BLOCKCHAIN\_ENV / FORKED\_LOCAL\_ENV\
  \-> account\[0]
* else (live chains: mainnet/rinkeby), \
  \-> use PK from config.yaml file

#### **In summary,**&#x20;

1. if index specified -> use ganache account
2. if id specified -> use id, stored account
3. none specified -> if local/forked env, use dummy, else use PK from config.yaml





