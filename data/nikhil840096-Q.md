### [Low] : `AiArenaHelper:constructor` while initializing the contract probablities are getting added two times in the `attributeProbabilities` mapping. one by this function `addAttributeProbabilities(0, probabilities)` and other by the for loop.


### [Low Finding]: AiArenaHelper:constructor potentially duplicates probabilities in the attributeProbabilities mapping during contract initialization.
**Description:** During contract initialization in the AiArenaHelper:constructor, probabilities may be inadvertently added twice to the attributeProbabilities mapping. Once via the function addAttributeProbabilities(0, probabilities) and again through the for loop.
```javascript
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
```
**Recommended Mitigation:** Ensure that probabilities are added only once during contract initialization by either utilizing a mapping or employing a for loop, but not both simultaneously.
```diff
    constructor(uint8[][] memory probabilities) {

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