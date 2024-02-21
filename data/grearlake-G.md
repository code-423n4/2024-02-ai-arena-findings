# 1, Duplicate work should be removed

## Lines of code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41-#L52

## Details
In the constructor, function `addAttributeProbabilities()` is called with `generation` = 0:

    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

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

Which mean `attributeProbabilities[0]`'s attribute is already is added, so re-add it is not necessary and wasting gas

## Recommendation
Remove following line:

    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
     -      attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    }