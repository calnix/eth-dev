# Yield Style Guide

## Use of underscore

Some style guide questions have come up wrt the use of `_` underscores with fn/var names.&#x20;

If you’ve read through many different contracts, I’m sure you’ve seen just as many different uses of the `_`  &#x20;

Some people always use them for fn params, some people use them for local vars. There is not one perfect approach.

The general rule that Alberto taught us (which also conforms to [Solidity style guide](https://docs.soliditylang.org/en/latest/style-guide.html#avoiding-naming-collisions)) is as follows:

**In general, avoid the use of `_` in variable or fn names.** <mark style="color:red;">There are only two exceptions:</mark>

1. If the var or fn name **shadows an existing name**, then use a trailing underscore `_`

```solidity
uint256 public someNumber;

function constructor(uint256 someNumber_) {
   someNumber = someNumber_;
}
```

2\. If the var or fn is visibility `private` or `internal` use a preceding `_`

```solidity
uint256 internal _internalVar;

function _internalCheck() internal {}
```

## Private vs internal

{% hint style="info" %}
* **internal** - only this contract and contracts deriving from it can access
* **private** - can be accessed only from this contract
{% endhint %}

* We generally don’t use `private` at all, **always favouring `internal`** .&#x20;
* It is really a question of security vs. flexibility. &#x20;

The arguments for using private over internal are mostly that we should use private in situations where we definitely never want any outside contract to call this fn/var and so don’t want to introduce a footgun. &#x20;

But it is just as likely you **may end up needing the var from an inheriting contract in an unknown way**.&#x20;

Conversely, the risk of an internal fn being overridden and causing a problem is really the responsibility of the code author. &#x20;

Who are we to limit their options? “Why not just copy paste the contract you’re inheriting from and change it however you want?” you might ask. &#x20;

**Because its nice to be able to cleanly import a fully audited contract and be done with it.**&#x20;

* If you start making your own versions of the audited contract then it becomes an unaudited contract, so we generally strive to not do that.&#x20;
* Furthermore, we strive to design contracts so that they can be flexible enough for someone to import it directly and use it as they see fit.&#x20;
* That someone is usually gonna be ourselves, but it’s also nice to think of the community at large when designing contracts that could be useful to others.
* This Yield policy is also in line with the [Solcurity Standard](https://github.com/Rari-Capital/solcurity#variables) which Yield has adopted and we will be looking at more carefully in subsequent assignments in the mentorship.&#x20;
