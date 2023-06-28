# Setting the TokenURI

create set\_tokenuri_._py in scripts/advcollectible

```python
from brownie import network
from brownie import AdvancedCollectible
from scripts.helpful_scripts import OpenSeaURL, get_account, get_breed

dog_metadata_dic = {
    "PUG": "https://ipfs.io/ipfs/Qmd9MCGtdVz2miNumBHDbvj8bigSgTwnr4SbyH6DNnpWdt?filename=0-PUG.json",
    "SHIBA_INU": "https://ipfs.io/ipfs/QmdryoExpgEQQQgJPoruwGJyZmz6SqV4FRTX1i73CT3iXn?filename=1-SHIBA_INU.json",
    "ST_BERNARD": "https://ipfs.io/ipfs/QmbBnUjyHHN7Ytq9xDsYF9sucZdDJLRkWz7vnZfrjMXMxs?filename=2-ST_BERNARD.json",
}

def main():
    print(f"Working on {network.show_active()}")
    advanced_collectible = AdvancedCollectible[-1]
    number_of_collectibles = advanced_collectible.tokenCounter() 
    print(f"You have {number_of_collectibles} tokenIds")

    for token_id in range(number_of_collectibles):
        breed = get_breed(advanced_collectible.tokenTobreed(token_id))
        # if it doesnt start with https, it has not been set
        if not advanced_collectible.tokenURI(token_id).startswith("https://"):
            print(f"Setting tokenURI of {token_id}")
            set_tokenURI(token_id,advanced_collectible, dog_metadata_dic[breed])


def set_tokenURI(token_id, nft_contract, tokenURI):
    account = get_account()
    tx = nft_contract.setTokenURI(token_id, tokenURI, {"from": account})
    tx.wait(1)

    print(f"Awesome! You can view your NFT at {OpenSeaURL.format(nft_contract.address, token_id)}")
    print("Wait up to 20 minutes, thne hit the refresh metadata buttons")

```

* Put most recent deployment of Advanced Collectible
* check number of minted NFTs
* Loop through the minted range of NFTs
* if tokenURI does not start with "http://" we know it has not been set,
  * and we proceed to set URI by calling set\_tokenURI()
