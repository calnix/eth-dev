# Mocking

1. Create a persistent mock network we can interact with
2. Add mock contracts

#### brownie runs on ganache-cli by default&#x20;

* spins up an instance of ganache chain and tears it down right after.
* cannot run secondary scripts after the initial script, as the universe has been torn down
* nothing saved into build/deployment folder

## Creating a persistent mock network

For persistence, deploy and run scripts on ganache-local. This allows for post-deployment contract interactions.

You can run secondary scripts and interact with deployed contracts -> fund\_withdraw.py

**Add ganache-local as persistent mock network:**

> brownie networks add **Ethereum** ganache-local host=http://127.0.0.1:8545 chainid=1337&#x20;

This will add ganache-local to the Ethereum category:&#x20;

![](<../../../.gitbook/assets/image (103).png>)

This requires that ganacheUI is running, so that brownie will latch onto it automatically to deploy required mock contracts.

Persistency will last till ganacheUI is closed.

{% hint style="info" %}
When you deploy a contract to your local ganache chain the first time, brownie stores addresses in its build folder. Those contracts live on your ganache chain.&#x20;

However, when you close ganacheUI and reopen, a new chain is created. Those contracts are gone, but brownie still thinks they exist.&#x20;

So references like fund\_me =FundMe\[-1], referring to most recent deployment might throw an error as the address is on a now non-existent mock chain.&#x20;

Either redeploy, or remove past deployments:\
\
To remove past deployments:

1. Either delete the build folder entirely,
2. Delete the contractt json in 1337 and the keys in map.json
{% endhint %}

### Update Deployment script

We need to update deploy.py since it identifies ganache-local as a live chain, and does not deploy mocks.

![](<../../../.gitbook/assets/image (288).png>)

The solution is to extend our definition of local blockchains by introducing variable into helpful\_scripts.py:

`LOCAL_BLOCKCHAIN_ENV = ["developments","ganache-local"]`

![](<../../../.gitbook/assets/image (251).png>)

Additionally, we will change get_accounts(),_ so that it uses a dummy account for _local\_blockchain_\_env

![](<../../../.gitbook/assets/image (291).png>)

import Local\__blockchain\_env into deploy.py and modif_y the if statement so it considers both development chains as well as the new ganache-local we created.&#x20;

![](<../../../.gitbook/assets/image (73).png>)

Finally we need to update our config file so that brownie knows that it should not verify on this chain.

Now on running deploy.py, we can see that the transactions are reflected in our ganache UI:

![](<../../../.gitbook/assets/image (248).png>)

## Creating Mock Contracts

Mocks are stored in contracts/test folder.&#x20;

> Download from:
>
> chainlink-mix / contracts / test / MockV3Aggregator.sol

Grab MockV3 and paste it into test folder.

{% hint style="warning" %}
To check if you got the correct mocks, run compile, after storing them.
{% endhint %}

> What if there is no mock that is readily available? Make your own

### MockV3Aggregator

Unlike its live deployment version, mock takes 2 parameters into its constructor:

* \#of decimals (we set at 8 d.p)
* initial Price (we set at 2000)
  * 200000000000 (2000 \*10\*\*8)

![](<../../../.gitbook/assets/image (109).png>)

![](<../../../.gitbook/assets/image (266).png>)

### Parameterize our contract address references

First we need to parameterize the hard-coded price-feed contract addresses in FundMe.sol

* create the interface as a state variable&#x20;
* initiate it with the constructor&#x20;
* use variable in getPrice functions

![FundMe.sol](<../../../.gitbook/assets/image (262).png>)

### Parameterize deployment function

now we need to pass the address variable as a constructor argument on deployment of FundMe.

![](<../../../.gitbook/assets/image (276).png>)

in brownie-config.yaml

![](<../../../.gitbook/assets/image (279).png>)

We cannot pre-store the address for the development chain mock, since it is different everytime. Instead we will use an if-else logic, to identify the netowrk we are on, and pass the correct address into price\__feed_\_address.

![instead of 18, it should be 8. so it resembles the aggregator price feed which returns 8 dp.](<../../../.gitbook/assets/image (286).png>)

### Parameterize verify&#x20;

![brownie-config.yaml](<../../../.gitbook/assets/image (111).png>)

modify .deploy:

![](<../../../.gitbook/assets/image (317).png>)

can use \["verify"], but if you forget to add it in it will throw index errors.

.get("verify") makes life easier that way.\


## Persistent Ganache deployment

Typically, brownie defaults to working with a local ganache-cli blockchain, when running our scripts. It will tear it back down after scripts have been completed.

* the usual one gets torn down immi after running deploy.py successfully.
* so we cannot interact with the deployed contracts or observe anything.

If we are running ganache UI, brownie will automatically attach itself to it and deploy there, as opposed to creating its own microcosm ganache-cli.

But brownie will not remember this deployment, as it is treated as a development chain.&#x20;

> There will be nothing saved down in the deployments folder.

![](<../../../.gitbook/assets/image (312).png>)

The deployed contracts json is saved into deployments folder. There are two: fundme.sol and our mock aggregator.&#x20;

We can interact with these deployed contracts persistently, via scripts.

### In summary we did 3 things

1. Add new network: ganache-local to Ethereum category (for persistency)
2. Add flag into helpful\_scripts.py: LOCAL\_BLOCKCHAIN\_ENV = \["developments","ganache-local"]
3. Modified our scripts to account for:
   1. get_accounts() -> pull dummy account_
   2. _ensure mocks are deployed_&#x20;
   3. _ensure verified is false_&#x20;

## WHAT DO WE NEED TO MOCK?

Think about Lottery.sol. It involves a number of external contracts:

* AggregatorV3Interface.sol -> chainlink ethusd pricefeed
* VRFConsumerBase.sol
* Ownable.sol
* VRF Coordinator
* LINK Token Contract

### Lottery.sol inherits Ownable and VRFConsumerBase&#x20;

* they are part of Lottery contract via extensibility.&#x20;
* "copying those codes into our contract"
* therefore **do not need mocks** since we inherit all their functions as if it were our own.

### Lottery.sol imports AggregatorV3Interface as an interface

* Interface is used to inform Lottery on how(via function definitions) it can access chainlink pricefeed
* Interface will require the mainnet/testnet address of the chainlink pricefeed contract.
* Therefore, we need to deploy a mock interface, that is self-reliant, and does not call out to another contract -> MockAggregatorV3 (it simply returns price as what we decide)
* Alternatively, we deploy a mock pricefeed contract and pass it through the interface&#x20;
  * (in this scenario you will have to create your own mock due to complexity of pricefeeds)

### Lottery.sol imports calls to VRFCoordinator&#x20;

* Lottery(inheriting VRFConsumerBase), will call VRFCoordinator to submit a request for randomness.
* To mimic this external transaction, we **need to mock** VRFCoordinator.

### Lottery has to pay fee in LINK Tokens (interact w/ LINK Token contract)

When requesting the VRFCoordinator for randomness, a fee in LINK tokens has to be paid.

* A transfer of LINK tokens, from Lottery.sol to VRFCoordinator has to be made as part of the request.
* Lottery.sol has to call transfer() on the LINK token contract to inform it to update its balances accordingly.
* **Need to mock Link Token Contract.**

{% hint style="info" %}
* To transfer link, we must call transfer() from Link Token contract.
* Token contracts are ledgers that track token balances across addresses
* so we must tell it: we sending our tokens to them
* same concept applies to all erc20 token contracts
{% endhint %}
