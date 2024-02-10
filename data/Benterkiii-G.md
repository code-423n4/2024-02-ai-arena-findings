
## the function variable should be declared before the array

the ´attributesLength´ should be declared before the ´finalAttributeProbabilityIndexes´ and use is instead of ´.length´ to save 12 gas every time the ´createPhysicalAttributes´ function get executed.

the array:
https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L96

the variable:
https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/AiArenaHelper.sol#L98

make the function look like this.

    <html>
      <head>
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
            uint256 attributesLength = attributes.length;

            uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributesLength);

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
      </head>
    </html>