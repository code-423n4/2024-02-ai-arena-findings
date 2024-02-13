### Neuron::claim()
The return value of transferFrom() is not check. It is a good practice to always check for return values for success and failure for token transfers.

### Neuron::mint()
The minting should be until max supply and not one less than total supply.

```
require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
```
The require condition should be 
```
require(totalSupply() + amount =< MAX_SUPPLY
```