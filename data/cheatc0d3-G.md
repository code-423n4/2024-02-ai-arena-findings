# AI Arena Gas Report

## Summary

| Issue | Severity | Occurrences |
|-------|----------|-------------|
| [[Gas] Prefer ERC721A over ERC721 for Gas Efficiency](#gas-prefer-erc721a-over-erc721-for-gas-efficiency) | Gas | 4 |
| [[Gas] Use uint256 for Boolean Operations to Save Gas](#gas-use-uint256-for-boolean-operations-to-save-gas) | Gas | 26 |
| [[Gas] Prefer Mappings Over Storage Arrays for Gas Efficiency](#gas-prefer-mappings-over-storage-arrays-for-gas-efficiency) | Gas | 30 |


### [Gas] Prefer ERC721A over ERC721 for Gas Efficiency
ERC721A is an optimized implementation of the ERC721 standard that reduces the gas cost for minting multiple NFTs in a single transaction. This pattern identifies contracts using ERC721 and suggests migrating to ERC721A.

#### Vulnerable Locations
- src/AAMintPass.sol:4
	```
	import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
	```
- src/AAMintPass.sol:5
	```
	import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
	```
- src/FighterFarm.sol:9
	```
	import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
	```
- src/FighterFarm.sol:10
	```
	import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
	```

#### Action Items
- Evaluate the feasibility of migrating from ERC721 to ERC721A in your project.
- Replace ERC721 imports with ERC721A and adjust the contract code to utilize ERC721A's optimized batch minting functionality.

#### References
- [https://github.com/chiru-labs/ERC721A](https://github.com/chiru-labs/ERC721A)

---

### [Gas] Use uint256 for Boolean Operations to Save Gas
Using uint256(1) and uint256(0) instead of true and false can save gas in certain operations. This pattern identifies places where boolean literals are used and suggests replacing them with their uint256 counterparts.

#### Vulnerable Locations
- src/AAMintPass.sol:34
	```
	true
	```
- src/AAMintPass.sol:49
	```
	true
	```
- src/AAMintPass.sol:57
	```
	false
	```
- src/AAMintPass.sol:59
	```
	true
	```
- src/AAMintPass.sol:67
	```
	true
	```
- src/AAMintPass.sol:74
	```
	false
	```
- src/FighterFarm.sol:141
	```
	true
	```
- src/GameItems.sol:98
	```
	true
	```
- src/GameItems.sol:152
	```
	false
	```
- src/GameItems.sol:154
	```
	true
	```
- src/GameItems.sol:187
	```
	true
	```
- src/MergingPool.sol:79
	```
	true
	```
- src/MergingPool.sol:130
	```
	true
	```
- src/Neuron.sol:73
	```
	true
	```
- src/RankedBattle.sol:156
	```
	true
	```
- src/RankedBattle.sol:248
	```
	false
	```
- src/RankedBattle.sol:254
	```
	true
	```
- src/RankedBattle.sol:262
	```
	true
	```
- src/RankedBattle.sol:281
	```
	true
	```
- src/RankedBattle.sol:282
	```
	true
	```
- src/RankedBattle.sol:286
	```
	false
	```
- src/RankedBattle.sol:433
	```
	false
	```
- src/RankedBattle.sol:435
	```
	true
	```
- src/RankedBattle.sol:489
	```
	false
	```
- src/RankedBattle.sol:470
	```
	true
	```
- src/VoltageManager.sol:54
	```
	true
	```

#### Action Items
- Replace true with uint256(1) and false with uint256(0) where applicable.
- Review the context of each replacement to ensure logical equivalence and that the optimization is beneficial.

#### References
- [https://github.com/devanshbatham/Solidity-Gas-Optimization-Tips](https://github.com/devanshbatham/Solidity-Gas-Optimization-Tips)

---

### [Gas] Prefer Mappings Over Storage Arrays for Gas Efficiency
Mappings are often more gas-efficient than storage arrays, especially for large datasets or when elements are frequently modified. This pattern identifies contracts using storage arrays where mappings could be more appropriate, taking into account the usage context to reduce false positives.

#### Vulnerable Locations
- src/AAMintPass.sol:112
	```
	string[] calldata _tokenURIs
	```
- src/AiArenaHelper.sol:17
	```
	string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"]
	```
- src/AiArenaHelper.sol:20
	```
	uint8[] public defaultAttributeDivisor = [2, 3, 5, 7, 11, 13]
	```
- src/AiArenaHelper.sol:30
	```
	mapping(uint256 => mapping(string => uint8[])) public attributeProbabilities
	```
- src/AiArenaHelper.sol:41
	```
	uint8[][] memory probabilities
	```
- src/AiArenaHelper.sol:68
	```
	uint8[] memory attributeDivisors
	```
- src/AiArenaHelper.sol:96
	```
	uint256[] memory finalAttributeProbabilityIndexes
	```
- src/AiArenaHelper.sol:131
	```
	uint8[][] memory probabilities
	```
- src/AiArenaHelper.sol:160
	```
	uint8[] memory
	```
- src/AiArenaHelper.sol:174
	```
	uint8[] memory attrProbabilities
	```
- src/FighterFarm.sol:69
	```
	FighterOps.Fighter[] public fighters
	```
- src/FighterFarm.sol:194
	```
	string[] calldata modelHashes
	```
- src/FighterFarm.sol:195
	```
	string[] calldata modelTypes
	```
- src/FighterFarm.sol:234
	```
	uint256[] calldata mintpassIdsToBurn
	```
- src/FighterFarm.sol:235
	```
	uint8[] calldata fighterTypes
	```
- src/FighterFarm.sol:236
	```
	uint8[] calldata iconsTypes
	```
- src/FighterFarm.sol:237
	```
	string[] calldata mintPassDnas
	```
- src/FighterFarm.sol:238
	```
	string[] calldata modelHashes
	```
- src/FighterFarm.sol:239
	```
	string[] calldata modelTypes
	```
- src/GameItems.sol:55
	```
	GameItemAttributes[] public allGameItemAttributes
	```
- src/MergingPool.sol:54
	```
	mapping(uint256 => address[]) public winnerAddresses
	```
- src/MergingPool.sol:123
	```
	address[] memory currentWinnerAddresses
	```
- src/MergingPool.sol:118
	```
	uint256[] calldata winners
	```
- src/MergingPool.sol:140
	```
	string[] calldata modelURIs
	```
- src/MergingPool.sol:141
	```
	string[] calldata modelTypes
	```
- src/MergingPool.sol:142
	```
	uint256[2][] calldata customAttributes
	```
- src/MergingPool.sol:206
	```
	uint256[] memory points
	```
- src/MergingPool.sol:205
	```
	uint256[] memory
	```
- src/Neuron.sol:127
	```
	address[] calldata recipients
	```
- src/Neuron.sol:127
	```
	uint256[] calldata amounts
	```

#### Action Items
- Evaluate the use of storage arrays in your project and consider replacing them with mappings where applicable.
- Review the data access patterns in your contract to determine if mappings offer a more gas-efficient alternative.

#### References
- [https://ethereum.stackexchange.com/questions/37549/array-or-mapping-which-costs-more-gas](https://ethereum.stackexchange.com/questions/37549/array-or-mapping-which-costs-more-gas)
