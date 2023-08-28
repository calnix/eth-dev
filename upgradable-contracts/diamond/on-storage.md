# On Storage



* Facets only read and write state variables in diamonds, not in themselves.
* Therefore, facets of a diamond share the same storage address space, that of the diamond.

**This can be  problem**

* For example let’s say that a diamond has two facets: FacetA and FacetB
* FacetA declares state variables `uint first` and `bytes32 second`&#x20;
* FacetB declares state variables `uint first` and `string name`

Both facets are storing `uint first` at position 0, so both facets can read and write that variable without problem. But both facets are writing and reading something different at position 1. They are messing up each other’s data because they are interpreting and writing different data at position 1.

{% hint style="warning" %}
This is why facets of the same diamond need to declare the same state variables in the same order.
{% endhint %}

Strategies need to exist to avoid storage collision, and make it easy for facets to declare the same state variables in the same order.&#x20;

### Inherited Storage

* One simple strategy is to create a contract (called 'Storage') that declares all state variables used by all the facets of a diamond.&#x20;
* It could then be inherited by every facet.

A limitation this has is that it prevents facets from being reusable. If you deploy a facet that uses Inherited Storage then you likely won’t be able to reuse that deployed facet with a different diamond that has different state variables.

{% hint style="info" %}
Another potential limitation, is that it is too easy to accidentally name something like an internal function or local variable the same name as a state variable and have a name clash. Especially with a large diamond with 100 or more state variables. But perhaps this could be overcome by using code naming conventions that prevent such name clashes.
{% endhint %}

### Diamond Storage

Different facets of the diamond to store their respective data at different locations. This avoids storage collision.

* Hash a unique string to get a random storage position and store a struct there.&#x20;
* The struct can contain all the state variables for a specific facet.&#x20;
* The unique string can act like a namespace for particular functionality.

#### Example: ERC721Facet

* This facet could store a struct called ‘`ERC721Storage`’ at position ‘`keccak256("com.myproject.erc721")`’.&#x20;
* The struct could contain all the state variables related to ERC721 that ERC721Facet reads and writes and nothing else.

**Advantages**

* **ERC721Facet is reusable.** ERC721Facet can be deployed only once, and the deployed ERC721Facet can be used with multiple different diamonds that are using facets with different state variables.
* ERC721Facet is not cluttered with state variable declarations of variables it doesn’t use.

{% hint style="info" %}
The current reference implementations of diamonds use Diamond Storage for the DiamondCutFacet, DiamondLoupeFacet and OwnershipFacet facets
{% endhint %}

{% hint style="info" %}
[https://dev.to/mudgen/how-diamond-storage-works-90e](https://dev.to/mudgen/how-diamond-storage-works-90e)
{% endhint %}

### App storage&#x20;

AppStorage is similar to Inherited Storage but it solves the name clash problem, where it is too easy to accidentally name something like an internal function or local variable the same name as a state variable.

* A struct called AppStorage is written in a Solidity file.&#x20;
* The AppStorage struct contains state variables that will be shared between facets.

{% hint style="info" %}
[https://dev.to/mudgen/appstorage-pattern-for-state-variables-in-solidity-3lki](https://dev.to/mudgen/appstorage-pattern-for-state-variables-in-solidity-3lki)
{% endhint %}

AppStorage is particularly useful for application or project specific facets that won’t be reused across different diamonds that use different storage. AppStorage can be used with Diamond Storage in the same facet, and this is common to do.\
