# AI Arena QA Report

## Summary

| Issue | Severity | Occurrences |
|-------|----------|-------------|
| [[Low] Division by Zero Not Prevented](#low-division-by-zero-not-prevented) | Low | 5 |
| [[Low] Check-Effects-Interaction Violation](#low-check-effects-interaction-violation) | Low | 2 |
| [[Low] Inefficient ABI Encoding for Non-Address Arguments](#low-inefficient-abi-encoding-for-non-address-arguments) | Low | 6 |
| [[NC] Use Caret in Pragma for Upgradeability](#nc-use-caret-in-pragma-for-upgradeability) | NC | 11 |
| [[Low] Missing Zero Address Check](#low-missing-zero-address-check) | Low | 28 |


### [Low] Division by Zero Not Prevented
Division by zero can occur when the divisor in a division operation is not checked for being zero before performing the division. This can lead to exceptions that may disrupt contract execution or result in incorrect logic being executed.

#### Vulnerable Locations
- src/AiArenaHelper.sol:107
	```
	dna / attributeToDnaDivisor[attributes[i]]
	```
- src/RankedBattle.sol:301
	```
	(
	                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
	            ) / totalAccumulatedPoints[currentRound]
	```
- src/RankedBattle.sol:392
	```
	(
	                accumulatedPointsPerAddress[claimer][i] * nrnDistribution
	            ) / totalAccumulatedPoints[i]
	```
- src/RankedBattle.sol:439
	```
	(bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4
	```
- src/RankedBattle.sol:528
	```
	(amountStaked[tokenId] + stakeAtRisk) / 10**18
	```

#### Action Items
- Ensure that all divisors in division operations are checked to be greater than zero before performing the division.
- Consider using SafeMath or a similar library that includes safe division operations.

#### References
- [https://docs.soliditylang.org/en/v0.8.0/control-structures.html#checked-or-unchecked-arithmetic](https://docs.soliditylang.org/en/v0.8.0/control-structures.html#checked-or-unchecked-arithmetic)

---

### [Low] Check-Effects-Interaction Violation
Functions that do not follow the check-effects-interaction pattern are potentially vulnerable to re-entrancy attacks.

#### Vulnerable Locations
- src/GameItems.sol:208
	```
	function createGameItem(
	        string memory name_,
	        string memory tokenURI,
	        bool finiteSupply,
	        bool transferable,
	        uint256 itemsRemaining,
	        uint256 itemPrice,
	        uint16 dailyAllowance
	    ) 
	        public 
	    {
	        require(isAdmin[msg.sender]);
	        allGameItemAttributes.push(
	            GameItemAttributes(
	                name_,
	                finiteSupply,
	                transferable,
	                itemsRemaining,
	                itemPrice,
	                dailyAllowance
	            )
	        );
	        if (!transferable) {
	          emit Locked(_itemCount);
	        }
	        setTokenURI(_itemCount, tokenURI);
	        _itemCount += 1;
	    }
	```
- src/RankedBattle.sol:322
	```
	function updateBattleRecord(
	        uint256 tokenId, 
	        uint256 mergingPortion,
	        uint8 battleResult,
	        uint256 eloFactor,
	        bool initiatorBool
	    ) 
	        external 
	    {   
	        require(msg.sender == _gameServerAddress);
	        require(mergingPortion <= 100);
	        address fighterOwner = _fighterFarmInstance.ownerOf(tokenId);
	        require(
	            !initiatorBool ||
	            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
	            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
	        );
	
	        _updateRecord(tokenId, battleResult);
	        uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
	        if (amountStaked[tokenId] + stakeAtRisk > 0) {
	            _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
	        }
	        if (initiatorBool) {
	            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
	        }
	        totalBattles += 1;
	    }
	```

#### Action Items
- Reorder the function code to perform all checks (e.g., require statements) before any state changes.
- Ensure that interactions (e.g., external calls) occur after all state changes.

#### References
- [https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html)
- [https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/#use-the-checks-effects-interactions-pattern](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/#use-the-checks-effects-interactions-pattern)

---

### [Low] Inefficient ABI Encoding for Non-Address Arguments
Instances where abi.encode() is used instead of abi.encodePacked() for non-address arguments can be less efficient.

#### Vulnerable Locations
- src/AAMintPass.sol:117
	```
	abi.encode(
	            msg.sender, 
	            numToMint[0], 
	            numToMint[1],
	            passesClaimed[msg.sender][0],
	            passesClaimed[msg.sender][1],
	            _tokenURIs
	        )
	```
- src/FighterFarm.sol:199
	```
	abi.encode(
	            msg.sender, 
	            numToMint[0], 
	            numToMint[1],
	            nftsClaimed[msg.sender][0],
	            nftsClaimed[msg.sender][1]
	        )
	```
- src/FighterFarm.sol:214
	```
	abi.encode(msg.sender, fighters.length)
	```
- src/FighterFarm.sol:254
	```
	abi.encode(mintPassDnas[i])
	```
- src/FighterFarm.sol:324
	```
	abi.encode(msg.sender, fighters.length)
	```
- src/FighterFarm.sol:379
	```
	abi.encode(msg.sender, tokenId, numRerolls[tokenId])
	```

#### Action Items
- Consider using abi.encodePacked() for encoding non-address arguments when efficiency is a concern and the data types do not introduce ambiguity.
- Ensure that the use of abi.encodePacked() does not lead to hash collisions or unexpected behavior.

#### References
- [https://docs.soliditylang.org/en/latest/abi-spec.html](https://docs.soliditylang.org/en/latest/abi-spec.html)
- [https://ethereum.stackexchange.com/questions/91826/why-are-there-two-methods-encoding-arguments-abi-encode-and-abi-encodepacked](https://ethereum.stackexchange.com/questions/91826/why-are-there-two-methods-encoding-arguments-abi-encode-and-abi-encodepacked)
- [https://forum.openzeppelin.com/t/abi-encode-vs-abi-encodepacked/2948](https://forum.openzeppelin.com/t/abi-encode-vs-abi-encodepacked/2948)

---

### [NC] Use Caret in Pragma for Upgradeability
Using the caret (^) symbol in the pragma directive allows for more flexible compiler versioning and can improve the upgradeability of the contract.

#### Vulnerable Locations
- src/AiArenaHelper.sol:3
	```
	pragma solidity >=0.8.0 <0.9.0;
	```
- src/FighterFarm.sol:2
	```
	pragma solidity >=0.8.0 <0.9.0;
	```
- src/FighterOps.sol:2
	```
	pragma solidity >=0.8.0 <0.9.0;
	```
- src/FixedPointMathLib.sol:2
	```
	pragma solidity >=0.8.0;
	```
- src/GameItems.sol:2
	```
	pragma solidity >=0.8.0 <0.9.0;
	```
- src/MergingPool.sol:2
	```
	pragma solidity >=0.8.0 <0.9.0;
	```
- src/Neuron.sol:2
	```
	pragma solidity >=0.8.0 <0.9.0;
	```
- src/RankedBattle.sol:2
	```
	pragma solidity >=0.8.0 <0.9.0;
	```
- src/StakeAtRisk.sol:2
	```
	pragma solidity >=0.8.0 <0.9.0;
	```
- src/Verification.sol:2
	```
	pragma solidity >=0.8.0 <0.9.0;
	```
- src/VoltageManager.sol:2
	```
	pragma solidity >=0.8.0 <0.9.0;
	```

#### Action Items
- Identify pragma directives that specify a fixed compiler version.
- Modify the pragma directive to use the caret (^) symbol for better upgradeability.

#### References
- [https://blog.solidityscan.com/understanding-solidity-pragma-and-its-security-practices-3b5458763a34?gi=69fb896a6948](https://blog.solidityscan.com/understanding-solidity-pragma-and-its-security-practices-3b5458763a34?gi=69fb896a6948)

---

### [Low] Missing Zero Address Check
In Solidity, contracts often interact with external addresses. Failing to check for a possible 0 address input (especially in constructors, setters, and initializer functions) before such interactions can lead to unexpected dangerous behavior. A zero address check ensures that addresses are explicitly provided and not left uninitialized or set to a default, invalid state.

#### Vulnerable Locations
- src/AAMintPass.sol:44
	```
	address _founderAddress
	```
- src/AAMintPass.sol:44
	```
	address _delegatedAddress
	```
- src/AAMintPass.sol:79
	```
	address _fighterFarmAddress
	```
- src/AAMintPass.sol:86
	```
	address _delegatedAddress
	```
- src/FighterFarm.sol:104
	```
	address ownerAddress
	```
- src/FighterFarm.sol:104
	```
	address delegatedAddress
	```
- src/FighterFarm.sol:104
	```
	address treasuryAddress_
	```
- src/FighterFarm.sol:171
	```
	address mergingPoolAddress
	```
- src/GameItems.sol:95
	```
	address ownerAddress
	```
- src/GameItems.sol:95
	```
	address treasuryAddress_
	```
- src/GameItems.sol:185
	```
	address newBurningAddress
	```
- src/MergingPool.sol:72
	```
	address ownerAddress
	```
- src/MergingPool.sol:73
	```
	address rankedBattleAddress
	```
- src/MergingPool.sol:74
	```
	address fighterFarmAddress
	```
- src/Neuron.sol:68
	```
	address ownerAddress
	```
- src/Neuron.sol:68
	```
	address treasuryAddress_
	```
- src/Neuron.sol:68
	```
	address contributorAddress
	```
- src/RankedBattle.sol:147
	```
	address ownerAddress
	```
- src/RankedBattle.sol:148
	```
	address gameServerAddress
	```
- src/RankedBattle.sol:149
	```
	address fighterFarmAddress
	```
- src/RankedBattle.sol:150
	```
	address voltageManagerAddress
	```
- src/RankedBattle.sol:184
	```
	address gameServerAddress
	```
- src/RankedBattle.sol:192
	```
	address stakeAtRiskAddress
	```
- src/StakeAtRisk.sol:61
	```
	address treasuryAddress_
	```
- src/StakeAtRisk.sol:62
	```
	address nrnAddress
	```
- src/StakeAtRisk.sol:63
	```
	address rankedBattleAddress
	```
- src/VoltageManager.sol:51
	```
	address ownerAddress
	```
- src/VoltageManager.sol:51
	```
	address gameItemsContractAddress
	```

#### Action Items
- Use require statements to validate addresses before any operation involving external addresses is performed, especially on constructors, setters or initializer functions.

#### References
- [https://detectors.auditbase.com/check-state-variable-address-zero-solidity](https://detectors.auditbase.com/check-state-variable-address-zero-solidity)