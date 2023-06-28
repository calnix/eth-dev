# NFT

1. SimpleCollectible&#x20;
2. AdvancedCollectible

## SimpleCollectible

{% tabs %}
{% tab title="SimpleCollectible.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.6.6;

// using Open Zepplin 3.x 
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";


contract SimpleCollectible is ERC721 {
    uint256 public tokenCounter;            //token ids will start frm 0

    // ERC20 constructor requires:
    // Name of NFT, symbol
    constructor() public ERC721("Doggie", "DOG") {
        tokenCounter = 0;              // just being explciit
    }

    
    function createCollectible(string memory _tokenURI) public returns(uint256) {  
        uint newTokenId = tokenCounter;
        
        _safeMint(msg.sender, newTokenId);
        _setTokenURI(newTokenId, _tokenURI);  //sets tokenURI for this tokenID - for images.

        tokenCounter = ++ tokenCounter;
        return newTokenId;          //return tokenID of the minted
    }

}
```



We inherit ERC721 (and its constructor):

* need to pass name and symbol as parameters

createCollectible() will be used to mint new NFTs by calling:

* \_safeMint() from ERC721
* \_setTokenURI() to pass the URI webstring&#x20;
* tokenCounter to track and increment NFTs minted
{% endtab %}

{% tab title="deploy.py" %}
```python
from brownie import SimpleCollectible
from scripts.helpful_scripts import get_account

sample_token_uri = "https://ipfs.io/ipfs/Qmd9MCGtdVz2miNumBHDbvj8bigSgTwnr4SbyH6DNnpWdt?filename=0-PUG.json"
OpenSeaURL = "https://testnets.opensea.io/assets/{}/{}"


def deploy():
    account = get_account()
    simple_collectible = SimpleCollectible.deploy({"from": account})

    # create/mint
    txt = simple_collectible.createCollectible(sample_token_uri)
    txt.wait(1)
    print(f"You can view the NFT at {OpenSeaURL.format(simple_collectible.address, simple_collectible.tokenCounter()-1)}")
    return simple_collectible

def main():
    deploy()
```

* sample json used as URI
* When deploying to rinkeby, the opensea link is accessible
*
{% endtab %}

{% tab title="test_simple_collectible.py" %}
```python
from brownie import SimpleCollectible
from brownie import network
from scripts.helpful_scripts import get_account, LOCAL_BLOCKCHAIN_ENV, FORKED_LOCAL_ENV
from scripts.deploy import deploy
import pytest

def test_can_create_simple_collectible():
    # Arrange
    if network.show_active() not in LOCAL_BLOCKCHAIN_ENV:
        pytest.skip()
    
    # Act
    simple_collectible = deploy()
    
    # Assert
    assert simple_collectible.tokenCounter() == 1 
    assert simple_collectible.ownerOf(0) == get_account()    
```

Assert:

* that 1 NFT was minted
* that the minter was the deployer account.
{% endtab %}
{% endtabs %}
