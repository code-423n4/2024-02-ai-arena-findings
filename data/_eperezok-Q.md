# Low-Risk Findings

## L-01: `MergingPool::getFighterPoints` will revert due to index out of bounds error

`MergingPool::getFighterPoints` retrieves the points for multiple fighters up to the specified maximum token ID.

```solidity
function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
    uint256[] memory points = new uint256[](1); // @audit-issue
    for (uint256 i = 0; i < maxId; i++) {
        points[i] = fighterPoints[i];
    }
    return points;
}
```

[MergingPool.sol#L205-L211](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L205-L211)

However, note that the `points` array it returns is initialized with length 1. Therefore, if `maxId` is set to something greater than 1, the call will revert with an index out of bounds error.

### Impact

`MergingPool::getFighterPoints` will only work for retrieving the points of a single fighter, which defeats its purpose.

### Recommended Mitigation Steps

Initialize the `points` array to be of length `maxId` instead of length `1`.

# Non-Critical Findings

## NC-01: attribute probabilities are set twice in the `AiArenaHelper` constructor

The attribute probabilities for generation 0 are set twice in the `AiArenaHelper` constructor.

```solidity
constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // @audit first time
    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
	// @audit second time
        attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
}
```

[AiArenaHelper.sol#L41-L52](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41-L52)

```solidity
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
    require(msg.sender == _ownerAddress);
    require(probabilities.length == 6, "Invalid number of attribute arrays");

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
        attributeProbabilities[generation][attributes[i]] = probabilities[i];
    }
}
```

[AiArenaHelper.sol#L131-L139](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131-L139)

### Impact

The gas cost of deploying the `AiArenaHelper` contract will be higher than it should.

### Recommended Mitigation Steps

Remove one of the two lines where the attribute probabilities are set.