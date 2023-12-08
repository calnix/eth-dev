# Testnet deployment

> brownie networks list

displays list of networks:

![](<../../../.gitbook/assets/image (174).png>)

### Deployment networks

![](<../../../.gitbook/assets/image (341).png>)

* these are temporary, like ganache networks.
* networks are torn down after running
* contracts and transactions sent on these networks are deleted after our script completes

### Connecting to a blockchain node

With remix we used MetaMask to connect to a proper testnet. In brownie, there is no such injection connection.

To connect to a blockchain node, we will use the RPC URL provided by infura or alchemy.&#x20;

* create a project in Infura
* go to project settings, and look for project ID:
* ![](<../../../.gitbook/assets/image (291).png>)
* in .env add the following line:
  * export WEB3\_INFURA\_PROJECT\_ID = cd3a18407dfe4d2597f756dafe2804d6



### Deploying to Rinkeby

> brownie run scripts/deploy.py --network rinkeby

![](<../../../.gitbook/assets/image (70).png>)

We can look up the transaction hashes and the contract address on rinkeby etherscan.
