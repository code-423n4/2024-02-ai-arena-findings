AiArenaHelper.sol#constructor initializes attributeProbabilities twice
```solidity
constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // Initialize the probabilities for each attribute
@>  addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
@>      attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
}
```

```solidity
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
    require(msg.sender == _ownerAddress);
    require(probabilities.length == 6, "Invalid number of attribute arrays");

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
@>      attributeProbabilities[generation][attributes[i]] = probabilities[i];
    }
 }
```

The relevant code is as shown above. In the `addAttributeProbabilities` function, `attributeProbabilities` is initialized once, and in the subsequent function flow, it is initialized again.