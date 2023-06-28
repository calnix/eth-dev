# Solidity Optimiser

* Number of optimizer runs is a trade-off between code size (deploy cost) and code execution cost
* A "runs" parameter of 1 will produce short but expensive code
* A larger runs parameter will produce longer but more gas efficient code\\
* Max value: 2\*\*32-1

{% hint style="info" %}
A common misconception is that this parameter specifies the number of iterations of the optimizer.&#x20;
{% endhint %}
