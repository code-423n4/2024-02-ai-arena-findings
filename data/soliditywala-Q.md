# 1
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

## 2
``attributeProbabilities`` in [AiArenaHelper](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol) is being set twice inside the constructor like below.
```solidity
  constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities); // @audit here  

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i]; // @audit here also
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```
## Recommendation
Remove the additional function call like below.

```solidity
  constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i]; // @audit here also
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```

## 3

[FighterOps.fighterCreatedEmitter()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L53) is callable by anyone and is used to spam the events.

