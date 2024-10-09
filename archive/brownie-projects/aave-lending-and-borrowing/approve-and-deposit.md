# Approve & Deposit

Aave `LendingPool` contract is the main contract of the protocol. It exposes all the user-oriented actions that can be invoked using either Solidity or web3 libraries.&#x20;

{% hint style="info" %}
[https://docs.aave.com/developers/v/2.0/the-core-protocol/lendingpool](https://docs.aave.com/developers/v/2.0/the-core-protocol/lendingpool)
{% endhint %}

* `LendingPool` contracts are upgradable. This means that their addresses may change in the future.&#x20;
* To prevent third party DApps from falling behind, Aave provides a `LendingPoolAddressesProvider` contract that will never be upgraded.&#x20;
* This is used to retrieve the latest `LendingPool`. As soon as we have the latest `LendingPool`, we can start depositing.

To interact with `ILendingPoolAddressesProvider` contract, we will need its address and ABI.&#x20;

## ILendingPoolAddressesProvider

#### ABI

To obtain the ABI, we will use an interface.&#x20;

Since we only require a select few functions: deposit, withdraw and so forth, we can create our own interface.

* create ILendingPoolAddressesProvider.sol in interfaces folder.
* either create your own interface by defining the functions (refer to etherscan or github)&#x20;
* ![](<../../../.gitbook/assets/image (304).png>)
* or, just copy aave's interfaces

#### Address

Addresses Provider -> Deployed Contracts Section

{% hint style="info" %}
[https://docs.aave.com/developers/v/2.0/deployed-contracts/deployed-contracts](https://docs.aave.com/developers/v/2.0/deployed-contracts/deployed-contracts)
{% endhint %}

![](<../../../.gitbook/assets/image (276).png>)

Add these to brownie-config.yaml

```python
def get_lendingpool():
    # create lending_pool_addressess_provider contract object
    lending_pool_addressess_provider = interface.ILendingPoolAddressesProvider(config["networks"][network.show_active()]["lending_pool_addressess_provider"])
    lending_pool = lending_pool_addressess_provider.getLendingPool()
    return lending_pool
```

## Lending Pool

#### ABI -> interface

* [https://docs.aave.com/developers/v/2.0/the-core-protocol/lendingpool/ilendingpool](https://docs.aave.com/developers/v/2.0/the-core-protocol/lendingpool/ilendingpool)
* Change import path from local directories to github

{% tabs %}
{% tab title="ILendingPool.sol" %}
Original:

```python
import {ILendingPoolAddressesProvider} from './ILendingPoolAddressesProvider.sol';
import {DataTypes} from './DataTypes.sol';
```

Modified:

```python
import {ILendingPoolAddressesProvider} from '@aave/contract/interfaces/ILendingPoolAddressesProvider.sol';
import {DataTypes} from '@aave/contract/protocol/libraries/types/DataTypes.sol';
```

compile after, to check if interfaces are correct.
{% endtab %}

{% tab title="brownie-config.yaml" %}
```yaml
dependencies:
  - aave/protocol-v2@1.0.1

compiler:
  solc:
    remappings:
      - '@aave=aave/protocol-v2@1.0.1'
```
{% endtab %}
{% endtabs %}

#### Address

We will get the address by calling getLendingPool() from the ILendingPoolAddressesProvider

```python
def get_lendingpool():
    # create lending_pool_addressess_provider contract object
    lending_pool_addressess_provider = interface.ILendingPoolAddressesProvider(config["networks"][network.show_active()]["lending_pool_addressess_provider"])
    #get lending pool address
    lending_pool = lending_pool_addressess_provider.getLendingPool()
    return lending_pool
```

### Code thus far

{% tabs %}
{% tab title="aave_borrow.py" %}
```python
from code import interact
from distutils.command.config import config
from scripts.helpful_scripts import get_account
from brownie import network, config, interface
from scripts.get_weth import get_weth

def main():
    account = get_account()
    deposit_token = config["networks"][network.show_active()]["weth_token"]
    # if no WETH, get_weth()
    # local mainnet fork can use dummy acocunts to get WETH
    # since local mainnet fork, can use dummy accounts. if actual mainnet/testnet then use private key account.
    if network.show_active() in ["mainnet-fork"]:
        get_weth()

    # Get lending pool contract
    lending_pool_address = get_lendingpool()
    lending_pool = interface.ILendingPool(lending_pool_address)
    print(f"....Lending pool contract: {lending_pool_address}....")

    # Approve sending our ERC20(WETH) tokens
    approve_erc20(deposit_amount, lending_pool.address, deposit_token, account)

      
def get_lendingpool():
    # create lending_pool_addressess_provider contract object
    lending_pool_addressess_provider = interface.ILendingPoolAddressesProvider(config["networks"][network.show_active()]["lending_pool_addressess_provider"])
    #get lending pool address
    lending_pool = lending_pool_addressess_provider.getLendingPool()
    return lending_pool
```

![](<../../../.gitbook/assets/image (261).png>)
{% endtab %}

{% tab title="helpful_scripts.py" %}
```python
from brownie import network,accounts,config

LOCAL_BLOCKCHAIN_ENV = ["development", "ganache-local"]
FORKED_LOCAL_ENV = ["mainnet-fork", "mainnet-fork-dev"]

def get_account(index=None,id=None):
    # accounts[0]  -- ganache accounts
    # accounts.add("env")  -- private key from env file ->  accounts.add(config["wallets"]["wallet1"])
    # accounts.load("id") -- load from brownie accounts list 
    if index:
        # if index was passed return ganache account
        return accounts[index]
    
    if id:
        return accounts.load(id)

    if network.show_active() in LOCAL_BLOCKCHAIN_ENV or network.show_active() in FORKED_LOCAL_ENV:
        return accounts[0]                                  #use ganache generated account.  
      
    else: #look in config.yaml
        return accounts.add(config["wallets"]["wallet1"])  

```
{% endtab %}

{% tab title="brownie-config.yaml" %}
```yaml
dependencies:
  - aave/protocol-v2@1.0.1

compiler:
  solc:
    remappings:
      - '@aave=aave/protocol-v2@1.0.1'

dotenv: .env

networks:
  default: development

  rinkeby:
    weth_token: '0xc778417E063141139Fce010982780140Aa0cD5Ab'  #WEthGateway contract
    lending_pool_addressess_provider: '0x88757f2f99175387ab4c6a4b3067c77a695b0349'
    verify: True

  kovan:
    weth_token: '0xd0A1E359811322d97991E03f863a0C30C2cF029C'  #WEthGateway contract
    lending_pool_addressess_provider: '0x88757f2f99175387ab4c6a4b3067c77a695b0349'
    verify: True
  
  mainnet-fork: #development env, fork uses live mainnet addresses 
    weth_token: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2'  #WEthGateway contract
    lending_pool_addressess_provider: '0xB53C1a33016B2DC2fF3653530bfF1848a515c8c5'

  mainnet: # for production
    weth_token: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2'  #WEthGateway contract

wallets:
  wallet1: ${PRIVATE_KEY
```
{% endtab %}
{% endtabs %}

## Approve

Before we can deposit we must approve Aave's contract to use our WETH tokens. This is done with the approve function.

![ERC20.sol](<../../../.gitbook/assets/image (345).png>)

{% hint style="info" %}
Because DApps (decentralized applications) use smart contracts to execute transactions, you must give permission for the smart contract to transfer up to a certain amount of your token (allowance).
{% endhint %}

To reiterate, transacting with ERC-20 tokens is a 2-step process:

1. Approval of token allowance
2. Submission of transaction&#x20;

### approve\_erc20()

{% tabs %}
{% tab title="approve_erc20" %}
```python
def approve_erc20(amount,spender, erc20_address, account):
    print("....Approving ERC20 token...")
    # get ERC20 interface: https://github.com/PatrickAlphaC/aave_brownie_py_freecode/tree/main/interfaces
    erc20 = interface.IERC20(erc20_address)

    # approve(address spender, uint256 value)
    tx = erc20.approve(spender,amount,{"from": account})
    tx.wait(1)
    print("....Approved....")
    return tx
```

In a generic implementation, we would create IERC20.sol in our interfaces folder, and pass the ERC20 token contract into the interface as above. \
\
Subsequently, we call on the token contract to approve setting our allowance for another 3rd party contract (via erc20.approve).
{% endtab %}

{% tab title="aave_borrow.py" %}
```python
from scripts.helpful_scripts import get_account
from brownie import network, config, interface
from scripts.get_weth import get_weth
from web3 import Web3

#0.1 ETH - 0.1*(10**18)
deposit_amount = Web3.toWei(0.1, "ether")  

def get_lendingpool():
    # create lending_pool_addressess_provider contract object
    lending_pool_addressess_provider = interface.ILendingPoolAddressesProvider(config["networks"][network.show_active()]["lending_pool_addressess_provider"])
    #get lending pool address
    lending_pool = lending_pool_addressess_provider.getLendingPool()
    return lending_pool

def approve_erc20(amount,spender, erc20_address, account):
    print("....Approving ERC20 token...")
    # get ERC20 interface: https://github.com/PatrickAlphaC/aave_brownie_py_freecode/tree/main/interfaces
    # approve(address spender, uint256 value)
    erc20 = interface.IERC20(erc20_address)
    tx = erc20.approve(spender,amount,{"from": account})
    tx.wait(1)
    print("....Approved....")
    return tx



def main():
    account = get_account()
    deposit_token = config["networks"][network.show_active()]["weth_token"]
    # if no WETH, get_weth()
    # local mainnet fork can use dummy acocunts to get WETH
    # since local mainnet fork, can use dummy accounts. if actual mainnet/testnet then use private key account.
    if network.show_active() in ["mainnet-fork"]:
        get_weth()

    # Get lending pool contract
    lending_pool_address = get_lendingpool()
    lending_pool = interface.ILendingPool(lending_pool_address)
    print(f"....Lending pool contract: {lending_pool_address}....")

    # Approve sending our ERC20(WETH) tokens
    approve_erc20(deposit_amount, lending_pool.address, deposit_token, account)
```
{% endtab %}
{% endtabs %}

Added to main():

* approve\_erc20(deposit\_amount, lending\_pool.address, deposit\_token, account)
* deposit\_amount as global variable

{% hint style="info" %}
since we are always runnning on mainnet-fork, set default = mainnet-fork in brownie-config.yaml, under the networks section.
{% endhint %}

## Deposit&#x20;

{% hint style="warning" %}
When depositing, the `LendingPool` contract must have**`allowance()`**to spend funds on behalf of**`msg.sender`** for at-least**`amount`** for the **`asset`** being deposited. This can be done via the standard ERC20 `approve()` method.
{% endhint %}

{% hint style="danger" %}
The referral program is currently inactive and you can pass`0` as the`referralCode.`

In future for referral code to be active again, a governance proposal, with the list of unique referral codes for various integration must be passed via governance.&#x20;
{% endhint %}

We will the below to main():

{% tabs %}
{% tab title="addition" %}
```python
## Deposit: deposit(address asset, uint256 amount, address onBehalfOf, uint16 referralCode)
# referralCode is deprecated - pass a 0 as parameter
print("....Depositing....")
tx = lending_pool.deposit(deposit_token_address, deposit_amount, account.address,0, {"from": account})
tx.wait(1) # wait one block
print("....Deposited!....")
```
{% endtab %}

{% tab title="aave_borrow.py" %}
```python
from scripts.helpful_scripts import get_account
from brownie import network, config, interface
from scripts.get_weth import get_weth
from web3 import Web3

#0.1 ETH - 0.1*(10**18)
deposit_amount = Web3.toWei(0.1, "ether")  

def get_lendingpool():
    # create lending_pool_addressess_provider contract object
    lending_pool_addressess_provider = interface.ILendingPoolAddressesProvider(config["networks"][network.show_active()]["lending_pool_addressess_provider"])
    #get lending pool address
    lending_pool = lending_pool_addressess_provider.getLendingPool()
    return lending_pool

def approve_erc20(amount,spender, erc20_address, account):
    print("....Approving ERC20 token...")
    # get ERC20 interface: https://github.com/PatrickAlphaC/aave_brownie_py_freecode/tree/main/interfaces
    # approve(address spender, uint256 value)
    erc20 = interface.IERC20(erc20_address)
    tx = erc20.approve(spender,amount,{"from": account})
    tx.wait(1)
    print("....Approved....")
    return tx


def main():
    account = get_account()
    deposit_token_address = config["networks"][network.show_active()]["weth_token"]
    # if no WETH, get_weth()
    # local mainnet fork can use dummy acocunts to get WETH
    # since local mainnet fork, can use dummy accounts. if actual mainnet/testnet then use private key account.
    if network.show_active() in ["mainnet-fork"]:
        get_weth()

    # Get lending pool contract
    lending_pool_address = get_lendingpool()
    lending_pool = interface.ILendingPool(lending_pool_address)
    print(f"....Lending pool contract: {lending_pool_address}....")

    # Approve sending our ERC20(WETH) tokens
    approve_erc20(deposit_amount, lending_pool.address, deposit_token_address, account)

    ## Deposit: deposit(address asset, uint256 amount, address onBehalfOf, uint16 referralCode)
    # referralCode is deprecated - just pass a 0 as parameter
    print("....Depositing....")
    tx = lending_pool.deposit(deposit_token_address, deposit_amount, account.address,0, {"from": account})
    tx.wait(1) # wait one block
    print("....Deposited!....")
```
{% endtab %}
{% endtabs %}
