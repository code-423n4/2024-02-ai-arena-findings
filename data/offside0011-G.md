
# [G-01] Pass empty strings for irrelevant parameters

`bytes("random")` can be replaced with `bytes("")`

```solidity
File: src\GameItems.sol
173:             _mint(msg.sender, tokenId, quantity, bytes("random"));

```

# [G-02] Downcasting cost extra gas

`dailyAllowanceReplenishTime` maps to uint256, the downcasting here cost more.

```solidity 
File: src\GameItems.sol
312:     function _replenishDailyAllowance(uint256 tokenId) private {
313:         allowanceRemaining[msg.sender][tokenId] = allGameItemAttributes[tokenId].dailyAllowance;
314:         dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
315:     }    

```

# [G-03] Inactive (unnecessary) function overriding

The following function overrides do nothing but cost extra gas.

```solidity
File: src\FighterFarm.sol
406:     /// @notice Returns whether a given interface is supported by this contract.
407:     /// @dev Calls ERC721.supportsInterface.
408:     /// @param _interfaceId The interface ID.
409:     /// @return Bool whether the interface is supported by this contract.
410:     function supportsInterface(bytes4 _interfaceId)
411:         public
412:         view
413:         override(ERC721, ERC721Enumerable)
414:         returns (bool)
415:     {
416:         return super.supportsInterface(_interfaceId);
417:     }

```

```solidity
File: src\FighterFarm.sol
443:     /// @notice Hook that is called before a token transfer.
444:     /// @param from The address transferring the token.
445:     /// @param to The address receiving the token.
446:     /// @param tokenId The ID of the NFT being transferred.
447:     function _beforeTokenTransfer(address from, address to, uint256 tokenId)
448:         internal
449:         override(ERC721, ERC721Enumerable)
450:     {
451:         super._beforeTokenTransfer(from, to, tokenId);
452:     }

```

# [G-04] Array length check is redundant

Solidity protects on array index overflow.

```solidity
File: src\FighterFarm.sol
243:         require(
244:             mintpassIdsToBurn.length == mintPassDnas.length && 
245:             mintPassDnas.length == fighterTypes.length && 
246:             fighterTypes.length == modelHashes.length &&
247:             modelHashes.length == modelTypes.length
248:         );

```

In this project, there appears to be an excessive use of array length checks. It's important to note that Solidity inherently prevents arrays from overflowing by default, and invalid inputs will automatically cause the function to revert. Therefore, these additional array length checks are unnecessary and can be safely removed.

# [G-05] Encompass additional logic

To optimize gas consumption, consider incorporating larger conditional statements that encompass additional logic.


```solidity
File: src\RankedBattle.sol
443:             /// If the user has no NRNs at risk, then they can earn points
444:             if (stakeAtRisk == 0) {
445:                 points = stakingFactor[tokenId] * eloFactor;
446:             }
447: 
448:             /// Divert a portion of the points to the merging pool
449:             uint256 mergingPoints = (points * mergingPortion) / 100;
450:             points -= mergingPoints;
451:             _mergingPoolInstance.addPoints(tokenId, mergingPoints);
```

use:

```solidity
/// If the user has no NRNs at risk, then they can earn points
if (stakeAtRisk == 0) {
    points = stakingFactor[tokenId] * eloFactor;
    /// Divert a portion of the points to the merging pool
    uint256 mergingPoints = (points * mergingPortion) / 100;
    points -= mergingPoints;
    _mergingPoolInstance.addPoints(tokenId, mergingPoints);
}
```



like:

```solidity
File: src\RankedBattle.sol
465:             /// Add points to the fighter for this round
466:             accumulatedPointsPerFighter[tokenId][roundId] += points;
467:             accumulatedPointsPerAddress[fighterOwner][roundId] += points;
468:             totalAccumulatedPoints[roundId] += points;
469:             if (points > 0) {
470:                 emit PointsChanged(tokenId, points, true);
471:             }

```

use:

```solidity
if (points > 0) {
    /// Add points to the fighter for this round
    accumulatedPointsPerFighter[tokenId][roundId] += points;
    accumulatedPointsPerAddress[fighterOwner][roundId] += points;
    totalAccumulatedPoints[roundId] += points;
    emit PointsChanged(tokenId, points, true);
}

```



like:


```solidity
File: src\RankedBattle.sol
485:                 accumulatedPointsPerFighter[tokenId][roundId] -= points;
486:                 accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
487:                 totalAccumulatedPoints[roundId] -= points;
488:                 if (points > 0) {
489:                     emit PointsChanged(tokenId, points, false);
490:                 }

```

use:

```solidity
if (points > 0) {
    accumulatedPointsPerFighter[tokenId][roundId] -= points;
    accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
    totalAccumulatedPoints[roundId] -= points;
    emit PointsChanged(tokenId, points, false);
}

```


# [G-06] Consider adding batchUpdateBattleRecord()

the current `updateBattleRecord()` is a frequently called funcion since every battle needs to call this once, which is very gas comsuming.

Maybe the admin needs to implement the `batchUpdateBattleRecord()` to save gas.

# [G-07] `fighterPoints` in MergingPool.sol has never been used for real logic

```solidity
File: src\MergingPool.sol
50:     /// @notice Mapping of address to fighter points.
51:     mapping(uint256 => uint256) public fighterPoints;
```

# [G-08] Redundant ERC20 tranfer check


the `allowance(treasuryAddress, msg.sender) >= amount` is redundant since `transferFrom` will takes care of this check.

```solidity
File: src\Neuron.sol
136:     /// @notice Claims the specified amount of tokens from the treasury address to the caller's address.
137:     /// @param amount The amount to be claimed
138:     function claim(uint256 amount) external {
139:         require(
140:             allowance(treasuryAddress, msg.sender) >= amount, 
141:             "ERC20: claim amount exceeds allowance"
142:         );
143:         transferFrom(treasuryAddress, msg.sender, amount);
144:         emit TokensClaimed(msg.sender, amount);
145:     }

```
