## [01] AiArenaHelper.sol the constructor initializes the attributeProbabilities() for generation 0 twice.
In the constructor of **`AiArenaHelper.sol`**, the initialization of **`attributeProbabilities()`** of generation 0 is duplicated.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L41-L52
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L137
```Solidity
constructor(uint8[][] memory probabilities) {
   _ownerAddress = msg.sender;

   // Initialize the probabilities for each attribute
@> addAttributeProbabilities(0, probabilities);

   uint256 attributesLength = attributes.length;
   for (uint8 i = 0; i < attributesLength; i++) {
@>      attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
} 
```
```Solidity
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
    require(msg.sender == _ownerAddress);
    require(probabilities.length == 6, "Invalid number of attribute arrays");

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
@>       attributeProbabilities[generation][attributes[i]] = probabilities[i];
    }
}
```
Remove one of the two initializations.