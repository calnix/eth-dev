# Aave - Lending and Borrowing

## Objectives&#x20;

1. Swap some ETH for WETH.
2. Deposit some WETH into Aave.
3. Borrow some asset against the collateral(deposit).
   1. Challenge: Sell that borrowed asset (short selling)
4. Repay everything back.

> aave testnet v2 is on kovan: [http://testnet.aave.com/](http://testnet.aave.com/)\
> aave testnet v3 is on rinkeby: [https://v3-test.aave.com/#/](https://v3-test.aave.com/#/)

### Testing

* Integration test: Kovan (v2) / Rinkeby (v3)&#x20;
* Unit tests: Mainnet-fork

If you are not working with oracles and don't need to mock off-chain responses, use a mainnet fork to run unit tests.&#x20;

## Deposit ETH

When you deposit ETH into AAVE, it gets swapped to WEth (ERC20 version of ETH) and then deposited.

{% hint style="info" %}
**Why must we convert to WETH?**

* Aave requires ERC20 properties of tokens for its ecosystem functionality.
* ETH does not conform to ERC-20.
* Wrapping ETH allows you to trade directly with Alt Tokens.

[https://weth.io/](https://weth.io/)
{% endhint %}

You will receive aETH which is the interest bearing token, reflective of your deposit and the interest it accrues.&#x20;

The Aave deposit function will handle all the necessary conversions from ETH. However, we can save on gas by locking our ETH for WETH directly with the WETHGateway contract.&#x20;

#### Create get\_weth()

Objective: Get WETH by depositing ETH \[call deposit function on WEth contract].

![](<../../../.gitbook/assets/image (37).png>)

* To interact with the contract, we need ABI + Address.&#x20;
* For ABI, use interface, IWEth.sol, from https://github.com/PatrickAlphaC/aave\_brownie\_py\_freecode/tree/main/interfaces
* create IWeth.sol in interfaces folder -> copy and paste github code into it.

![](<../../../.gitbook/assets/image (248).png>)

{% tabs %}
{% tab title="get_weth().py" %}
```python
from brownie import config, network, interface 
from scripts.helpful_scripts import get_account


def get_weth():
    """ 
    how to get Weth?
    deposit eth to WETH contract, it gives us ETH
    https://kovan.etherscan.io/token/0xd0a1e359811322d97991e03f863a0c30c2cf029c#writeContract
    
    to interact with this contract we need: address + ABI
    ABI: use interface, IWEth.sol,  from https://github.com/PatrickAlphaC/aave_brownie_py_freecode/tree/main/interfaces
    create IWeth.sol in interfaces folder -> copy and paste github code into it.
    
    """
    account = get_account()
    weth = interface.IWeth(config["networks"][network.show_active()]["weth_token"])

    #deposit 0.1 ETH, should get back 0.1 WETH
    tx = weth.deposit({"from": account, "value": 0.1 * 10**18})
    tx.wait(1)
    print("...Received 0.1WETH...")
    return tx

     
def main():
    get_weth()
```
{% endtab %}

{% tab title="brownie-config.yaml" %}
```yaml
dotenv: .env

networks:
  default: development

  rinkeby:
    weth_token: '0xc778417E063141139Fce010982780140Aa0cD5Ab'  #WEthGateway contract
    verify: True

  kovan:
    weth_token: '0xd0A1E359811322d97991E03f863a0C30C2cF029C'  #WEthGateway contract
    verify: True
  
  mainnet-fork: #development env, fork uses live mainnet addresses 
    weth_token: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2'  #WEthGateway contract
  
  mainnet: # for production
    weth_token: '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2'  #WEthGateway contract

wallets:
  wallet1: ${PRIVATE_KEY}
```
{% endtab %}

{% tab title="helpful-scripts.py" %}
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
{% endtabs %}

On running get\_weth.py successfully, you should see 0.1 WEth in your Metamask wallet.&#x20;

![](<../../../.gitbook/assets/image (307).png>)

{% hint style="info" %}
If you want to convert Weth back to Eth, call the withdraw function. The function will return us the ETH we stored with the contract, on reclaiming the WETH.
{% endhint %}

![](<../../../.gitbook/assets/image (84).png>)
