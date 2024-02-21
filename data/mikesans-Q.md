The `deleteAttributeProbabilities` function of `AiArenaHelper.sol` does not delete `attributeProbabilities[generation]` but rather sets the arrays to be empty arrays of length 0. It would be more proper to use `delete`.

**Example**
*Github* : https://github.com/code-423n4/2024-02-ai-area/blob/main/src/AiArenaHelper.sol#L144-151

**Suggested Revision**
I suggest the following change:


```solidity
    function deleteAttributeProbabilities(uint8 generation) public {
        require(msg.sender == _ownerAddress);

        uint256 attributesLength = attributes.length;
        delete attributeProbabilities[generation];
    }
```