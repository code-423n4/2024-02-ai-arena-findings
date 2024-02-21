# Unnecessary Loop for Gas Efficiency in `AiArenaHelper::constructor`

## Impact

The line `attributeProbabilities[0][attributes[i]] = probabilities[i]` essentially repeats the same operation as `addAttributeProbabilities(0, probabilities)`, which cause unnecessary loop of gas consumption.

## Proof of Concept

The function `addAttributeProbabilities(0, probabilities)` has already set up the initial values for the generation of the `addAttributeProbabilities` mapping based on the provided probabilities for attributes.

The line `attributeProbabilities[0][attributes[i]] = probabilities[i]` essentially repeats the same operation as `addAttributeProbabilities(0, probabilities)`, where it assigns probabilities to attributes for the initial generation. Therefore, the `addAttributeProbabilities()` function is redundant and can be removed to reduce unnecessary loops for gas usage optimization, as it doesn't provide any additional functionality beyond what's already been established by the line `attributeProbabilities[0][attributes[i]] = probabilities[i]`. 

[AiArenaHelper::constructor](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L45C1-L45C53)
[AiArenaHelper::addAttributeProbabilities](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L137C1-L137C82)

```solidity

    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;
+       require(probabilities.length == 6, "Invalid number of attribute arrays");

-       // Initialize the probabilities for each attribute
-       addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
} 

    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
}



```

## Tools Used
Manual Review

## Recommendation
It's imperative to remove the redundant function `addAttributeProbabilities(0, probabilities)` to to reduce unnecessary loops for gas usage optimization as shown in the part of Proof of Concept

```solidity

    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;
+       require(probabilities.length == 6, "Invalid number of attribute arrays");

-       // Initialize the probabilities for each attribute
-       addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
} 

    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
}