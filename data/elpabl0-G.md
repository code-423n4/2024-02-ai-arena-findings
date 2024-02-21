## Redundant `AiArenaHelper.sol::addAttributeProbabilities` Call in `Constructor` Increases Gas Usage

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L41C1-L52C7

The deployment cost with the `addAttributeProbabilities` call is `1,741,279`, while the cost without this call is `1,660,491`. Therefore, it's recommended to remove this function call to reduce gas usage. Instead, consider adding a `require` statement to validate the input.

```diff
       constructor(uint8[][] memory probabilities) {
+       require(probabilities.length == 6, "Invalid number of attribute arrays");
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
-        addAttributeProbabilities(0, probabilities); 

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    }
```