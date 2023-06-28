# Compact Strings

EVM stores data in 32 byte 'buckets' or 'slots'.

Strings are arrays of UTF-8 characters, and arrays are basically a fixed length sequence of storage slots located next to each other. Therefore, like everything else we need to think in terms of storage buckets

**This means that when we compile our contracts, we would prefer to use a single slot for the strings we use.**

Each character in a string is a UTF-8 Encoded byte, meaning that your strings can be up to 32 characters in length if you do not use specially encoded characters like emojis to be contained in a single storage slot.

{% hint style="info" %}
Here's a great resource to look up how many bytes a string takes up under UTF-8 Encoding. [https://mothereff.in/byte-counter](https://mothereff.in/byte-counter)\
[https://blog.hubspot.com/website/what-is-utf-8](https://blog.hubspot.com/website/what-is-utf-8)
{% endhint %}
