# X && Y, ||

### X && Y

* if X is false, it doesn't matter what Y is
* x > 2 && y > 3  -> if the first condition is false, the second will not get evaluated
* this is because the entire expression will evaluate to false
* Implication: put the cheaper condition first, so there is a possibility to skip the evaluation of the second&#x20;

### X || Y

* only one condition needs to be true
* put the cheaper condition to evaluate first
