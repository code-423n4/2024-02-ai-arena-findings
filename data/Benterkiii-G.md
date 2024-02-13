# Improvements to `createPhysicalAttributes` Function

 **Variable Declaration Order:**
The function variable `attributesLength` should be declared before the array `finalAttributeProbabilityIndexes`.

```solidity
   uint256 attributesLength = attributes.length;
   uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributesLength);
```
by doing this we save 12 gas every time the function get executed.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L96

Updated function:

```solidity
function createPhysicalAttributes(
        uint256 dna, 
        uint8 generation, 
        uint8 iconsType, 
        bool dendroidBool
    ) 
        external 
        view 
        returns (FighterOps.FighterPhysicalAttributes memory) 
    {
        if (dendroidBool) {
            return FighterOps.FighterPhysicalAttributes(99, 99, 99, 99, 99, 99);
        } else {
            uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);

            uint256 attributesLength = attributes.length;
            for (uint8 i = 0; i < attributesLength; i++) {
                if (
                  i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
                  i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
                  i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
                ) {
                    finalAttributeProbabilityIndexes[i] = 50;
                } else {
                    uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;
                    uint256 attributeIndex = dnaToIndex(generation, rarityRank, attributes[i]);
                    finalAttributeProbabilityIndexes[i] = attributeIndex;
                }
            }
            return FighterOps.FighterPhysicalAttributes(
                finalAttributeProbabilityIndexes[0],
                finalAttributeProbabilityIndexes[1],
                finalAttributeProbabilityIndexes[2],
                finalAttributeProbabilityIndexes[3],
                finalAttributeProbabilityIndexes[4],
                finalAttributeProbabilityIndexes[5]
            );
        }
    }
```

# Functions whom being called inside the Contract should default to internal

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L67
```solidity
function getFighterAttributes(Fighter storage self) public view returns (uint256[6] memory) {
        return [
            self.physicalAttributes.head,
            self.physicalAttributes.eyes,
            self.physicalAttributes.mouth,
            self.physicalAttributes.body,
            self.physicalAttributes.hands,
            self.physicalAttributes.feet
        ];
}
