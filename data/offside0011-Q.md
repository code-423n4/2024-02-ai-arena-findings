# [01] missing document of `generation` @param

```solidity
File: src\AiArenaHelper.sol
165:      /// @dev Convert DNA and rarity rank into an attribute probability index.
166:      /// @param attribute The attribute name.
167:      /// @param rarityRank The rarity rank.
168:      /// @return attributeProbabilityIndex attribute probability index.
169:     function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
```


# [02] Inadequate randomness

`dna` generation is controlled by `msg.sender`. User can fuzz the sender address to gain extra privilege by generating more competetive `dna`s.

```solidity
File: src\FighterFarm.sol
379:             uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
380:             (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
381:             fighters[tokenId].element = element;
382:             fighters[tokenId].weight = weight;
383:             fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
384:                 newDna,
385:                 generation[fighterType],
386:                 fighters[tokenId].iconsType,
387:                 fighters[tokenId].dendroidBool
388:             );

```