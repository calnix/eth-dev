# Brownie basics

To have brownie deploy the folder structure within a folder:

`brownie init`

### Deploy.py

This file will handle deployment logic. yyIn your deploy.py file, in scripts folder:

```python
# for blockchain deplyment
# brownie run <scriptname.py>

#brownie has an accounts package
from brownie import accounts 

#put all the deployment logic in one function:
def deploy_simple_storage():
    #local ganache accounts
    account = accounts[0]; print(account)

    #use own account for testnet
    #add account to brownie: brownie accounts new <acc_name:freecodecamp-account>


def main():
    deploy_simple_storage()
```

### Accounts management

Brownie has a package called accounts for handling accounts. In terminal:

```bash
brownie accounts new freecodecamp-account
```

Terminal will request for your private key:

![](<../../../.gitbook/assets/image (106).png>)

Add 0x to the front of the private key before pasting it in. Brownie will password encrypt the private key.

#### accounts list

> &#x20;\> brownie accounts list

![](<../../../.gitbook/assets/image (225).png>)

this will list all the accounts user has added.

#### delete accounts&#x20;

&#x20;\> brownie accounts delete \<acc\_name>
