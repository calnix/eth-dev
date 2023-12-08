# Create Metadata

This assumes there already was a previous deployment to rinkeby, with the utilization of `AdvancedCollectible[-1]`

### Setup

* create new folder "metadata" in brownie project root
* create new file: sample\_metadata.py
* create new folder within metadata, "rinkeby"

{% code title="sample_metadata.py" %}
```python
metadata_template = {
    "name": "",
    "description": "",
    "image": "",
    "attributes": [{"trait_type": "cuteness", "value": 100}]
}
```
{% endcode %}

![](<../../.gitbook/assets/image (326).png>)

### create\_metadata.py

In our scripts folder, create new file: create\_metadata.py

We will use this script to assign the URI for each NFT.

{% hint style="info" %}
talk about NFT token URI etc - what, how
{% endhint %}

{% code title="create_metadata.py" %}
```python
from brownie import AdvancedCollectible, network
from metadata.sample_metadata import metadata_template
from pathlib import Path

breed_mapping = {0: "PUG", 1: "SHIBA_INU", 2: "ST_BERNARD"}

def get_breed(breed_number):
    return breed_mapping[breed_number]


def main():
    advanced_collectible = AdvancedCollectible[-1]
    number_of_NFTS = advanced_collectible.tokenCounter()
    print(f"The number of NFTs minted so far is {number_of_NFTS}")

    for tokenID in range(number_of_NFTS):
        breed = get_breed(advanced_collectible.tokenIdToBreed(tokenID))
        metadata_filename = f"./metadata/{network.show_active()}/{tokenID}-{breed}.json"
    
        collectible_metadata = metadata_template
        if Path(metadata_filename).exists():
            print(f"{metadata_filename} exists! Delete to overwrite")
        else:
            print(f"Creating Metadata file: {metadata_filename}")
            collectible_metadata["name"] = breed
            collectible_metadata["description"] = f"An adorable {breed} pup!"
            print(collectible_metadata)

            image_path = "./img" + breed.lower().replace("_", "-") + ".png"
            image_uri = upload_to_ipfs()
            collectible_metadata["image_uri"] = image_uri

```
{% endcode %}

* call on previous rinkeby deployment
* check number of NFTs minted
* for each tokenID, get the breed via get_breed(),_ and structure the filename and path as metadata\_filename

![metadata\_filenamecollectible\_metadata](<../../.gitbook/assets/image (315).png>)

* collectible\_metadata collects the metadata\_template to be used as a base for modification
* We then check if the required metadata\_filename exists, using Path library.&#x20;
* Else, we will create a new one using the template as reference, adding the following
  * name
  * description
  * image URI ->  collectible\_metadata\["image\_uri"] = image\_uri

### image-uri

How do we get the image URI? Currently the images are in our local img folder. We need them to be hosted on IPFS and pass their IPFS URI into collectible\_metadata\["image\_uri"]&#x20;

To that end, we will have to install IPFS cli:[ ](https://docs.ipfs.io/how-to/command-line-quick-start/)[https://docs.ipfs.io/how-to/command-line-quick-start/](https://docs.ipfs.io/how-to/command-line-quick-start/)

{% hint style="info" %}
Interestingly, ipfs daemon command works in terminal, command prompt.
{% endhint %}

On running ipfs daemon on terminal we will see the following:

![](<../../.gitbook/assets/image (100).png>)

We will be working with **/api/v0/add** to add our file to IPFS: [https://docs.ipfs.io/reference/http/api/#http-rpc-commands](https://docs.ipfs.io/reference/http/api/#http-rpc-commands)

{% tabs %}
{% tab title="upload_to_ipfs(filepath)" %}
```python
def upload_to_ipfs(filepath):
    # open image as binary - open(rb)
    with Path(filepath).open("rb") as fp:
        image_binary = fp.read()
        ipfs_url = "http://127.0.0.1:5001"     #get from WebUI 
        endpoint = "/api/v0/add"
        response = requests.post(ipfs_url + endpoint, files={"file":image_binary}) #post request
        ipfs_hash = response.json()["Hash"]     #response returns dictionary
        # "./img/0-PUG.png"  -> split on /, grab last part of array, which is "0-PUG.png"
        filename = filepath.split("/")[-1:][0]
        image_uri = f"https://ipfs.io/ipfs/{ipfs_hash}?filename={filename}"
        print(image_uri)
        return image_uri
```
{% endtab %}

{% tab title="def main" %}
```python
def main():
    advanced_collectible = AdvancedCollectible[-1]
    number_of_NFTS = advanced_collectible.tokenCounter()
    print(f"The number of NFTs minted so far is {number_of_NFTS}")

    for tokenID in range(number_of_NFTS):
        breed = get_breed(advanced_collectible.tokenIdToBreed(tokenID))
        metadata_filename = f"./metadata/{network.show_active()}/{tokenID}-{breed}.json"
    
        collectible_metadata = metadata_template
        if Path(metadata_filename).exists():
            print(f"{metadata_filename} exists! Delete to overwrite")
        else:
            print(f"Creating Metadata file: {metadata_filename}")
            collectible_metadata["name"] = breed
            collectible_metadata["description"] = f"An adorable {breed} pup!"
            print(collectible_metadata)
            # convert underscores to dashes to be URI compatible
            image_path = "./img/" + breed.lower().replace("_", "-") + ".png"
            image_uri = upload_to_ipfs(image_path)
         #   collectible_metadata["image_uri"] = image_uri
```
{% endtab %}
{% endtabs %}

* ipfs\_url -> retrieved from running ipds daemon
* endpoint -> as per api endpoint reference
* response -> structuring and sending a post response (can also use curl)
* ipfs\_hash -> response returns JSON string on successful call to endpoint - decode it as a python dict via .json()

![reponse returns a dictionary](<../../.gitbook/assets/image (230).png>)

* extract filename from filepath and together with the IPFS hash, construct the image URI

{% hint style="info" %}
More on JSON: [https://www.usna.edu/Users/cs/nchamber/courses/forall/s20/lec/l25/#:\~:text=JSON%20at%20its%20top%2Dlevel,%2C%20other%20dictionaries%2C%20and%20lists.](https://www.usna.edu/Users/cs/nchamber/courses/forall/s20/lec/l25/)
{% endhint %}

#### print(image\_uri)

will return an ipfs link, like so:

[https://ipfs.io/ipfs/QmUPjADFGEKmfohdTaNcWhp7VGk26h5jXDA7v3VtTnTLcW?filename=st-bernard.png](https://ipfs.io/ipfs/QmUPjADFGEKmfohdTaNcWhp7VGk26h5jXDA7v3VtTnTLcW?filename=st-bernard.png) &#x20;

{% hint style="warning" %}
The image will be available as long as you are running your ipfs node.

To make the data highly available without needing to run a local IPFS daemon 24/7, you can request that a remote pinning service store a copy of your IPFS data on their IPFS nodes.
{% endhint %}

#### Creating metadate files

Now that we have filled all the fields in the metadata file, we are going to write it as a json file into our directory.

```python
        with open(metadata_filename, "w") as file:
            json.dump(collectible_metadata, file)
        upload_to_ipfs(metadata_filename)
```

This will create a json file in our metadata/rinkeby folder upon running:

`brownie run scripts/advcollectible/create_metadata.py --network rinkeby`

![](<../../.gitbook/assets/image (86).png>)

#### Final code at this point:

```python
from brownie import AdvancedCollectible, network
from metadata.sample_metadata import metadata_template
from pathlib import Path
import requests, json

breed_mapping = {0: "PUG", 1: "SHIBA_INU", 2: "ST_BERNARD"}

def get_breed(breed_number):
    return breed_mapping[breed_number]


def main():
    advanced_collectible = AdvancedCollectible[-1]
    number_of_NFTS = advanced_collectible.tokenCounter()
    print(f"The number of NFTs minted so far is {number_of_NFTS}")

    for tokenID in range(number_of_NFTS):
        breed = get_breed(advanced_collectible.tokenIdToBreed(tokenID))
        metadata_filename = f"./metadata/{network.show_active()}/{tokenID}-{breed}.json"
    
        collectible_metadata = metadata_template
        if Path(metadata_filename).exists():
            print(f"{metadata_filename} exists! Delete to overwrite")
        else:
            print(f"Creating Metadata file: {metadata_filename}")
            collectible_metadata["name"] = breed
            collectible_metadata["description"] = f"An adorable {breed} pup!"
            print(collectible_metadata)
            # convert underscores to dashes to be URI compatible
            image_path = "./img/" + breed.lower().replace("_", "-") + ".png"
            image_uri = upload_to_ipfs(image_path)
            collectible_metadata["image_uri"] = image_uri
            
            with open(metadata_filename, "w") as file:
                json.dump(collectible_metadata, file)
            upload_to_ipfs(metadata_filename)


def upload_to_ipfs(filepath):
    # open image as binary - opne(rb)
    with Path(filepath).open("rb") as fp:
        image_binary = fp.read()
        ipfs_url = "http://127.0.0.1:5001"     #get from WebUI 
        endpoint = "/api/v0/add"
        response = requests.post(ipfs_url + endpoint, files={"file": image_binary})       #post request
        ipfs_hash = response.json()["Hash"]     #response returns dictionary. 
        # "./img/0-PUG.png"  -> split on /, grab last part of array, which is "0-PUG.png"
        filename = filepath.split("/")[-1:][0]
        image_uri = f"https://ipfs.io/ipfs/{ipfs_hash}?filename={filename}"
        print(image_uri)
        return image_uri   
```

{% hint style="info" %}
11:33 - Refactor to check if file has already been uploaded to IPFS, so we don't upload every single time. (check github for code)
{% endhint %}

### Alternative: Pinata

deploy to Pinata -> upload\_pinata.py

Reference: [https://docs.pinata.cloud/api-pinning/pin-file](https://docs.pinata.cloud/api-pinning/pin-file)   \


* create account at Pinata and generate api key
* 11:20 - 11:31



![](<../../.gitbook/assets/image (250).png>)

You should also see the pinned file:

![](<../../.gitbook/assets/image (26).png>)
