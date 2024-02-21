### Duplicate execution of the same code

You shouldn't call `addAttributeProbabilities(0, probabilities)` in the constructor on line 45, because you add the attribute probabilities manually on line 49:

*Instances (1)*:

```solidity
File: src/FighterFarm.sol

41:    constructor(uint8[][] memory probabilities) {
42:        _ownerAddress = msg.sender;
43:
44:        // Initialize the probabilities for each attribute
45:        addAttributeProbabilities(0, probabilities);
46:
47:        uint256 attributesLength = attributes.length;
48:        for (uint8 i = 0; i < attributesLength; i++) {
49:            attributeProbabilities[0][attributes[i]] = probabilities[i];
50:            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
51:        }
52:    }

```
```solidity

131:    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
132:        require(msg.sender == _ownerAddress);
133:        require(probabilities.length == 6, "Invalid number of attribute arrays");
134:
135:        uint256 attributesLength = attributes.length;
136:        for (uint8 i = 0; i < attributesLength; i++) {
137:            attributeProbabilities[generation][attributes[i]] = probabilities[i];
138:        }
139:    }

```

[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol)
