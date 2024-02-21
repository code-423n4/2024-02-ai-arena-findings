## L-01 The max supply of NRN tokens will always be less than the amount defined in the `MAX_SUPPLY` constant 

In the mint function of `Neuron.sol` due to [this](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L156) check, the max amount of NRN that can be minted will atleast be 1 less than the intented amount defined in the [MAX_SUPPLY constant](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L43).

## Recommendation

Currently the check is
```solidity
require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
```

consider making the change as
```solidity
require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply");
```

