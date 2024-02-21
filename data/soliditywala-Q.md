Following condition in [Neuron.mint()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L156) is wrong when enforcing the MAX_SUPPLY.
```solidity
 require(
            totalSupply() + amount < MAX_SUPPLY,
            "Trying to mint more than the max supply"
        );
```
It allows to mint ``MAX_SUPPLY - 1`` due to ``<`` which is 1 less than ``MAX_SUPPLY``

## Recommendation

Use ``<=`` instead of ``<`` like below.
```solidity
 require(
            totalSupply() + amount <= MAX_SUPPLY,
            "Trying to mint more than the max supply"
        );
```