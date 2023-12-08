# verify and publish

rinkeby etherscan > go to contract address > ![](<../../../.gitbook/assets/image (351).png>)

![](<../../../.gitbook/assets/image (323).png>)

Then we will come to this screen:

![](<../../../.gitbook/assets/image (320).png>)

We need to input the flattened solidity source code -> meaning copypasta all the imports and so forth.

can pro-grammatically flatten:

* get API key from ehterscan account
* input into brownie in .env file:

![](<../../../.gitbook/assets/image (80).png>)

* in deploy.py, edit your deploy statement like so:

![](<../../../.gitbook/assets/image (147).png>)

now rerun brownie run deploy.py --network rinkeby

![](<../../../.gitbook/assets/image (349).png>)
