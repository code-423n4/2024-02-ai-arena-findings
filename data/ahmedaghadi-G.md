## 1. Code of `addAttributeProbabilities` function is repeated in `AiArenaHelper` contract's constructor even after calling the function.

The `AiArenaHelper` constructor ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41 ):

```solidity
/// @dev Constructor to initialize the contract with the attribute probabilities for gen 0.
/// @param probabilities An array of attribute probabilities for the generation.
constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // Initialize the probabilities for each attribute
@->    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
@->        attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
}
```

`attributeProbabilities[0][attributes[i]] = probabilities[i];` is already done in `addAttributeProbabilities` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131 ):

```solidity
/// @notice Add attribute probabilities for a given generation.
/// @dev Only the owner can call this function.
/// @param generation The generation number.
/// @param probabilities An array of attribute probabilities for the generation.
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
    require(msg.sender == _ownerAddress);
    require(probabilities.length == 6, "Invalid number of attribute arrays");

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
@->        attributeProbabilities[generation][attributes[i]] = probabilities[i];
    }
}
```

So the steps of `addAttributeProbabilities` function is repeated in `AiArenaHelper` contract's constructor even after calling the function. Gas can be saved by removing the redundant steps in the constructor:

```diff
constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // Initialize the probabilities for each attribute
    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
-        attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
}
```

## 2. `fighters.length` is a storage variable which is used in a loop thus can be cached outside the loop.

The `FighterFarm::claimFighters` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L191 ):

```solidity
function claimFighters(
    uint8[2] calldata numToMint,
    bytes calldata signature,
    string[] calldata modelHashes,
    string[] calldata modelTypes
)
    external
{
    bytes32 msgHash = bytes32(keccak256(abi.encode(
        msg.sender,
        numToMint[0],
        numToMint[1],
        nftsClaimed[msg.sender][0],
        nftsClaimed[msg.sender][1]
    )));
    require(Verification.verify(msgHash, signature, _delegatedAddress));
    uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
    require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
    nftsClaimed[msg.sender][0] += numToMint[0];
    nftsClaimed[msg.sender][1] += numToMint[1];
    for (uint16 i = 0; i < totalToMint; i++) {
        _createNewFighter(
            msg.sender,
@->            uint256(keccak256(abi.encode(msg.sender, fighters.length))),
            modelHashes[i],
            modelTypes[i],
            i < numToMint[0] ? 0 : 1,
            0,
            [uint256(100), uint256(100)]
        );
    }
}
```

Here, `_createNewFighter` function will increment the `fighters.length` as new `fighter` will be pushed in `fighters` array.
So we can cache the `fighters.length`, which is used in the loop, before the loop and increment the cached value inside the loop to save gas:

```diff
function claimFighters(
    uint8[2] calldata numToMint,
    bytes calldata signature,
    string[] calldata modelHashes,
    string[] calldata modelTypes
)
    external
{
    bytes32 msgHash = bytes32(keccak256(abi.encode(
        msg.sender,
        numToMint[0],
        numToMint[1],
        nftsClaimed[msg.sender][0],
        nftsClaimed[msg.sender][1]
    )));
    require(Verification.verify(msgHash, signature, _delegatedAddress));
    uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
    require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
    nftsClaimed[msg.sender][0] += numToMint[0];
    nftsClaimed[msg.sender][1] += numToMint[1];
+    uint256 fightersLength = fighters.length;
    for (uint16 i = 0; i < totalToMint; i++) {
        _createNewFighter(
            msg.sender,
-            uint256(keccak256(abi.encode(msg.sender, fighters.length))),
+            uint256(keccak256(abi.encode(msg.sender, fightersLength))),
            modelHashes[i],
            modelTypes[i],
            i < numToMint[0] ? 0 : 1,
            0,
            [uint256(100), uint256(100)]
        );
+        unchecked{++fightersLength};
    }
}
```

## 3. Storage variable used more than once in a function can be cached.

<details>

<summary>
The instances having this issue:

</summary>

###

#

```
File: `FighterFarm.sol`

244            mintpassIdsToBurn.length == mintPassDnas.length &&
245            mintPassDnas.length == fighterTypes.length &&
246            fighterTypes.length == modelHashes.length &&
247            modelHashes.length == modelTypes.length

249        for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
```

`mintpassIdsToBurn.length`, `mintPassDnas.length`, `fighterTypes.length`, `modelHashes.length`, `modelTypes.length` are used more than once in the function.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L233

#

```
File: `FighterFarm.sol`

372    require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);

378    numRerolls[tokenId] += 1;

379    uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
```

`numRerolls[tokenId]` is used more than once in the function.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L370

#

```
File: `GameItems.sol`
159    dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp ||

166    if (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp) {
```

`dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp` is used more than once in the function.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L147

#

```
File: `GameItems.sol`
160    quantity <= allowanceRemaining[msg.sender][tokenId]

169    allowanceRemaining[msg.sender][tokenId] -= quantity;
```

`allowanceRemaining[msg.sender][tokenId]` is used more than once in the function.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L147

#

```
File: `GameItems.sol`
152    allGameItemAttributes[tokenId].finiteSupply == false ||

154    allGameItemAttributes[tokenId].finiteSupply == true &&

170    if (allGameItemAttributes[tokenId].finiteSupply) {
```

`allGameItemAttributes[tokenId].finiteSupply` is used more than once in the function.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L147

#

```
File: `GameItems.sol`
155    quantity <= allGameItemAttributes[tokenId].itemsRemaining

171    allGameItemAttributes[tokenId].itemsRemaining -= quantity;
```

`allGameItemAttributes[tokenId].itemsRemaining` is used more than once in the function.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L147

#

```
File: `GameItems.sol`
231    emit Locked(_itemCount);

233    setTokenURI(_itemCount, tokenURI);

234    _itemCount += 1;
```

`_itemCount` is used more than once in the function.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L208

#

```
File: `GameItems.sol`
295    require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");

299    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
```

`_itemCount` is used more than once in the function.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L294

</details>

## 4. No need to check for `msg.sender` to be owner before `burn` function as it will revert if the caller is not the owner.

In the `redeemMintPass` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L233 ):

```solidity
function redeemMintPass(
    uint256[] calldata mintpassIdsToBurn,
    uint8[] calldata fighterTypes,
    uint8[] calldata iconsTypes,
    string[] calldata mintPassDnas,
    string[] calldata modelHashes,
    string[] calldata modelTypes
)
    external
{
    require(
        mintpassIdsToBurn.length == mintPassDnas.length &&
        mintPassDnas.length == fighterTypes.length &&
        fighterTypes.length == modelHashes.length &&
        modelHashes.length == modelTypes.length
    );
    for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
@->        require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
@->        _mintpassInstance.burn(mintpassIdsToBurn[i]);
        _createNewFighter(
            msg.sender,
            uint256(keccak256(abi.encode(mintPassDnas[i]))),
            modelHashes[i],
            modelTypes[i],
            fighterTypes[i],
            iconsTypes[i],
            [uint256(100), uint256(100)]
        );
    }
}
```

The require check `require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));` isn't required as ` _mintpassInstance.burn(mintpassIdsToBurn[i]);` will revert if `msg.sender` isn't the owner of the `mintpassIdsToBurn[i]`. So the require check can be removed.

## 5. Reducing the condition in `require` check will save the gas.

In the `mint` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L147 ):

```solidity
function mint(uint256 tokenId, uint256 quantity) external {
    require(tokenId < _itemCount);
    uint256 price = allGameItemAttributes[tokenId].itemPrice * quantity;
    require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");
@->    require(
        allGameItemAttributes[tokenId].finiteSupply == false ||
        (
            allGameItemAttributes[tokenId].finiteSupply == true &&
            quantity <= allGameItemAttributes[tokenId].itemsRemaining
        )
    );
    require(
        dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp ||
        quantity <= allowanceRemaining[msg.sender][tokenId]
    );

    _neuronInstance.approveSpender(msg.sender, price);
    bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, price);
    if (success) {
        if (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp) {
            _replenishDailyAllowance(tokenId);
        }
        allowanceRemaining[msg.sender][tokenId] -= quantity;
        if (allGameItemAttributes[tokenId].finiteSupply) {
            allGameItemAttributes[tokenId].itemsRemaining -= quantity;
        }
        _mint(msg.sender, tokenId, quantity, bytes("random"));
        emit BoughtItem(msg.sender, tokenId, quantity);
    }
}
```

The condition in `require( allGameItemAttributes[tokenId].finiteSupply == false || (allGameItemAttributes[tokenId].finiteSupply == true && quantity <= allGameItemAttributes[tokenId].itemsRemaining) );` can be reduced to `allGameItemAttributes[tokenId].finiteSupply == false || quantity <= allGameItemAttributes[tokenId].itemsRemaining`.

It is due to the fact that `a or (!a and b)` is equivalent to `a or b`. Which can also be verified on https://web.stanford.edu/class/cs103/tools/truth-table-tool/ or any other truth table generator.

- Truth table for `a or (!a and b)`:

    <table><tr><th>a</th><th>b</th><th>(a ∨ (¬a ∧ b))</th></tr><tr><td>F</td><td>F</td><td>F</td></tr><tr><td>F</td><td>T</td><td>T</td></tr><tr><td>T</td><td>F</td><td>T</td></tr><tr><td>T</td><td>T</td><td>T</td></tr></table>

- Truth table for `a or b`:

    <table><tr><th>a</th><th>b</th><th>(a ∨ b)</th></tr><tr><td>F</td><td>F</td><td>F</td></tr><tr><td>F</td><td>T</td><td>T</td></tr><tr><td>T</td><td>F</td><td>T</td></tr><tr><td>T</td><td>T</td><td>T</td></tr></table>

## 6. `!isSelectionComplete[roundId]` isn't required to check in the `MergingPool::pickWinner` function.

In the `MergingPool::pickWinner` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L118 ):

```solidity
function pickWinner(uint256[] calldata winners) external {
    require(isAdmin[msg.sender]);
    require(winners.length == winnersPerPeriod, "Incorrect number of winners");
@->    require(!isSelectionComplete[roundId], "Winners are already selected");
    uint256 winnersLength = winners.length;
    address[] memory currentWinnerAddresses = new address[](winnersLength);
    for (uint256 i = 0; i < winnersLength; i++) {
        currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
        totalPoints -= fighterPoints[winners[i]];
        fighterPoints[winners[i]] = 0;
    }
    winnerAddresses[roundId] = currentWinnerAddresses;
    isSelectionComplete[roundId] = true;
    roundId += 1;
}
```

The `require(!isSelectionComplete[roundId], "Winners are already selected");` check isn't required as every time `MergingPool::pickWinner` function is called, `roundId` is incremented so `isSelectionComplete[roundId]` will always be false.

## 7. No need to add or subtract with `points` if it's zero in `RankedBattle::_addResultPoints`.

The `_addResultPoints` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L416 ):

```solidity
function _addResultPoints(
    uint8 battleResult,
    uint256 tokenId,
    uint256 eloFactor,
    uint256 mergingPortion,
    address fighterOwner
)
    private
{
    ...

    if (battleResult == 0) {
        /// If the user won the match

        ...

        /// Add points to the fighter for this round
        accumulatedPointsPerFighter[tokenId][roundId] += points;
        accumulatedPointsPerAddress[fighterOwner][roundId] += points;
        totalAccumulatedPoints[roundId] += points;
        if (points > 0) {
            emit PointsChanged(tokenId, points, true);
        }
    } else if (battleResult == 2) {
        /// If the user lost the match

        ...

        if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
            /// If the fighter has a positive point balance for this round, deduct points

            ...

            accumulatedPointsPerFighter[tokenId][roundId] -= points;
            accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
            totalAccumulatedPoints[roundId] -= points;
            if (points > 0) {
                emit PointsChanged(tokenId, points, false);
            }
        } else {
            /// If the fighter does not have any points for this round, NRNs become at risk of being lost

            ...

        }
    }
}
```

There's already an `if` check for `points > 0` before adding or subtracting `points` to `accumulatedPointsPerFighter[tokenId][roundId]`, `accumulatedPointsPerAddress[fighterOwner][roundId]`, and `totalAccumulatedPoints[roundId]`. So there's no need to add or subtract with `points` if it's zero.

The changes can be made as follows:

```diff
function _addResultPoints(
    uint8 battleResult,
    uint256 tokenId,
    uint256 eloFactor,
    uint256 mergingPortion,
    address fighterOwner
)
    private
{
    ...

    if (battleResult == 0) {
        /// If the user won the match

        ...

        /// Add points to the fighter for this round
-        accumulatedPointsPerFighter[tokenId][roundId] += points;
-        accumulatedPointsPerAddress[fighterOwner][roundId] += points;
-        totalAccumulatedPoints[roundId] += points;
        if (points > 0) {
+            accumulatedPointsPerFighter[tokenId][roundId] += points;
+            accumulatedPointsPerAddress[fighterOwner][roundId] += points;
+            totalAccumulatedPoints[roundId] += points;
            emit PointsChanged(tokenId, points, true);
        }
    } else if (battleResult == 2) {
        /// If the user lost the match

        ...

        if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
            /// If the fighter has a positive point balance for this round, deduct points

            ...

-            accumulatedPointsPerFighter[tokenId][roundId] -= points;
-            accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
-            totalAccumulatedPoints[roundId] -= points;
            if (points > 0) {
+                accumulatedPointsPerFighter[tokenId][roundId] -= points;
+                accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
+                totalAccumulatedPoints[roundId] -= points;
                emit PointsChanged(tokenId, points, false);
            }
        } else {
            /// If the fighter does not have any points for this round, NRNs become at risk of being lost

            ...

        }
    }
}
```

## 8. Updating `curStakeAtRisk` in `RankedBattle::_addResultPoints` function only in block where it's used can save gas if that block isn't reached.

The `_addResultPoints` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L416 ):

```solidity
function _addResultPoints(
    uint8 battleResult,
    uint256 tokenId,
    uint256 eloFactor,
    uint256 mergingPortion,
    address fighterOwner
)
    private
{
    ...

@-> if (battleResult == 0) {
        /// If the user won the match

        ...

        /// Do not allow users to reclaim more NRNs than they have at risk
@->     if (curStakeAtRisk > stakeAtRisk) {
            curStakeAtRisk = stakeAtRisk;
        }

        /// If the user has stake-at-risk for their fighter, reclaim a portion
        /// Reclaiming stake-at-risk puts the NRN back into their staking pool
@->     if (curStakeAtRisk > 0) {
            _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
            amountStaked[tokenId] += curStakeAtRisk;
        }

        ...

@-> } else if (battleResult == 2) {
        /// If the user lost the match

        /// Do not allow users to lose more NRNs than they have in their staking pool
@->     if (curStakeAtRisk > amountStaked[tokenId]) {
            curStakeAtRisk = amountStaked[tokenId];
        }
        if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
            /// If the fighter has a positive point balance for this round, deduct points

            ...

@->     } else {
            /// If the fighter does not have any points for this round, NRNs become at risk of being lost
            bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
            if (success) {
                _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                amountStaked[tokenId] -= curStakeAtRisk;
            }
        }
    }
}
```

For `battleResult == 0`, the `if` block checking `curStakeAtRisk > stakeAtRisk` can be added inside the next if block for `curStakeAtRisk > 0` as `curStakeAtRisk` is only used in that block. This will save gas if the `if` block isn't reached.

Similarly, for `battleResult == 2`, the `if` block checking `curStakeAtRisk > amountStaked[tokenId]` can be added inside the `else` case of the next `if` block for `accumulatedPointsPerFighter[tokenId][roundId] > 0` as `curStakeAtRisk` is only used in that block. This will save gas if the `else` block isn't reached.

The changes can be made as follows:

```diff
function _addResultPoints(
    uint8 battleResult,
    uint256 tokenId,
    uint256 eloFactor,
    uint256 mergingPortion,
    address fighterOwner
)
    private
{
    ...

    if (battleResult == 0) {
        /// If the user won the match

        ...

-        /// Do not allow users to reclaim more NRNs than they have at risk
-        if (curStakeAtRisk > stakeAtRisk) {
-            curStakeAtRisk = stakeAtRisk;
-        }

        /// If the user has stake-at-risk for their fighter, reclaim a portion
        /// Reclaiming stake-at-risk puts the NRN back into their staking pool
        if (curStakeAtRisk > 0) {
+            /// Do not allow users to reclaim more NRNs than they have at risk
+            if (curStakeAtRisk > stakeAtRisk) {
+                curStakeAtRisk = stakeAtRisk;
+            }
            _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
            amountStaked[tokenId] += curStakeAtRisk;
        }

        ...

    } else if (battleResult == 2) {
        /// If the user lost the match

-        /// Do not allow users to lose more NRNs than they have in their staking pool
-        if (curStakeAtRisk > amountStaked[tokenId]) {
-            curStakeAtRisk = amountStaked[tokenId];
-        }
        if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
            /// If the fighter has a positive point balance for this round, deduct points

            ...

        } else {
+            /// Do not allow users to lose more NRNs than they have in their staking pool
+            if (curStakeAtRisk > amountStaked[tokenId]) {
+                curStakeAtRisk = amountStaked[tokenId];
+            }
            /// If the fighter does not have any points for this round, NRNs become at risk of being lost
            bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
            if (success) {
                _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                amountStaked[tokenId] -= curStakeAtRisk;
            }
        }
    }
}
```

## 9. No need to consider `points` if `stakeAtRisk != 0` in `RankedBattle::_addResultPoints`.

The `_addResultPoints` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L416 ):

```solidity
function _addResultPoints(
    uint8 battleResult,
    uint256 tokenId,
    uint256 eloFactor,
    uint256 mergingPortion,
    address fighterOwner
)
    private
{
    uint256 stakeAtRisk;
    uint256 curStakeAtRisk;
@-> uint256 points = 0;

    ...

    if (battleResult == 0) {
        /// If the user won the match

        /// If the user has no NRNs at risk, then they can earn points
@->     if (stakeAtRisk == 0) {
            points = stakingFactor[tokenId] * eloFactor;
        }

        /// Divert a portion of the points to the merging pool
        uint256 mergingPoints = (points * mergingPortion) / 100;
        points -= mergingPoints;
        _mergingPoolInstance.addPoints(tokenId, mergingPoints);

        ...

        /// Add points to the fighter for this round
        accumulatedPointsPerFighter[tokenId][roundId] += points;
        accumulatedPointsPerAddress[fighterOwner][roundId] += points;
        totalAccumulatedPoints[roundId] += points;
        if (points > 0) {
            emit PointsChanged(tokenId, points, true);
        }
    } else if (battleResult == 2) {
        /// If the user lost the match

        ...

    }
}
```

The `points` is initialized to `0` and then it's updated only if `stakeAtRisk == 0` for `battleResult == 0` `if` block, . So there's no need to consider `points` if `stakeAtRisk != 0`. So the following changes can be made:

```diff
function _addResultPoints(
    uint8 battleResult,
    uint256 tokenId,
    uint256 eloFactor,
    uint256 mergingPortion,
    address fighterOwner
)
    private
{
    uint256 stakeAtRisk;
    uint256 curStakeAtRisk;
    uint256 points = 0;

    ...

    if (battleResult == 0) {
        /// If the user won the match

        /// If the user has no NRNs at risk, then they can earn points
        if (stakeAtRisk == 0) {
            points = stakingFactor[tokenId] * eloFactor;
+
+            /// Divert a portion of the points to the merging pool
+            uint256 mergingPoints = (points * mergingPortion) / 100;
+            points -= mergingPoints;
+            _mergingPoolInstance.addPoints(tokenId, mergingPoints);
+
+            /// Add points to the fighter for this round
+            accumulatedPointsPerFighter[tokenId][roundId] += points;
+            accumulatedPointsPerAddress[fighterOwner][roundId] += points;
+            totalAccumulatedPoints[roundId] += points;
+            if (points > 0) {
+                emit PointsChanged(tokenId, points, true);
+            }
        }

-        /// Divert a portion of the points to the merging pool
-        uint256 mergingPoints = (points * mergingPortion) / 100;
-        points -= mergingPoints;
-        _mergingPoolInstance.addPoints(tokenId, mergingPoints);

        ...

-        /// Add points to the fighter for this round
-        accumulatedPointsPerFighter[tokenId][roundId] += points;
-        accumulatedPointsPerAddress[fighterOwner][roundId] += points;
-        totalAccumulatedPoints[roundId] += points;
-        if (points > 0) {
-            emit PointsChanged(tokenId, points, true);
-        }
    } else if (battleResult == 2) {
        /// If the user lost the match

        ...

    }
}
```

## 10. Storage variable used as counter inside the loop can be updated after the loop.

The `RankedBattle::claimNRN` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L294 ):

```solidity
function claimNRN() external {
    require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
    uint256 claimableNRN = 0;
    uint256 nrnDistribution;
    uint32 lowerBound = numRoundsClaimed[msg.sender];
    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
        nrnDistribution = getNrnDistribution(currentRound);
        claimableNRN += (
            accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution
        ) / totalAccumulatedPoints[currentRound];
@->        numRoundsClaimed[msg.sender] += 1;
    }
    if (claimableNRN > 0) {
        amountClaimed[msg.sender] += claimableNRN;
        _neuronInstance.mint(msg.sender, claimableNRN);
        emit Claimed(msg.sender, claimableNRN);
    }
}
```

The `MergingPool::claimRewards` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L139 ):

```solidity
function claimRewards(
    string[] calldata modelURIs,
    string[] calldata modelTypes,
    uint256[2][] calldata customAttributes
)
    external
{
    uint256 winnersLength;
    uint32 claimIndex = 0;
    uint32 lowerBound = numRoundsClaimed[msg.sender];
    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
@->        numRoundsClaimed[msg.sender] += 1;
        winnersLength = winnerAddresses[currentRound].length;
        for (uint32 j = 0; j < winnersLength; j++) {
            if (msg.sender == winnerAddresses[currentRound][j]) {
                _fighterFarmInstance.mintFromMergingPool(
                    msg.sender,
                    modelURIs[claimIndex],
                    modelTypes[claimIndex],
                    customAttributes[claimIndex]
                );
                claimIndex += 1;
            }
        }
    }
    if (claimIndex > 0) {
        emit Claimed(msg.sender, claimIndex);
    }
}
```

In both the functions, the storage variable `numRoundsClaimed[msg.sender]` is used as a counter inside the loop and is updated inside the loop. It can be updated after the loop to save gas.

The changes can be made as follows for `claimNRN` function:

```diff
function claimNRN() external {
    require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
    uint256 claimableNRN = 0;
    uint256 nrnDistribution;
    uint32 lowerBound = numRoundsClaimed[msg.sender];
    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
        nrnDistribution = getNrnDistribution(currentRound);
        claimableNRN += (
            accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution
        ) / totalAccumulatedPoints[currentRound];
-        numRoundsClaimed[msg.sender] += 1;
    }
+    numRoundsClaimed[msg.sender] = roundId;
    if (claimableNRN > 0) {
        amountClaimed[msg.sender] += claimableNRN;
        _neuronInstance.mint(msg.sender, claimableNRN);
        emit Claimed(msg.sender, claimableNRN);
    }
}
```

The changes can be made as follows for `claimRewards` function:

```diff
function claimRewards(
    string[] calldata modelURIs,
    string[] calldata modelTypes,
    uint256[2][] calldata customAttributes
)
    external
{
    uint256 winnersLength;
    uint32 claimIndex = 0;
    uint32 lowerBound = numRoundsClaimed[msg.sender];
    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
-        numRoundsClaimed[msg.sender] += 1;
        winnersLength = winnerAddresses[currentRound].length;
        for (uint32 j = 0; j < winnersLength; j++) {
            if (msg.sender == winnerAddresses[currentRound][j]) {
                _fighterFarmInstance.mintFromMergingPool(
                    msg.sender,
                    modelURIs[claimIndex],
                    modelTypes[claimIndex],
                    customAttributes[claimIndex]
                );
                claimIndex += 1;
            }
        }
    }
+    numRoundsClaimed[msg.sender] = roundId;
    if (claimIndex > 0) {
        emit Claimed(msg.sender, claimIndex);
    }
}
```

## 11. Variable initialized only in constructor can be made immutable.

In the `RankedBattle` constructor ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L146 ):

```solidity
constructor(
    address ownerAddress,
    address gameServerAddress,
    address fighterFarmAddress,
    address voltageManagerAddress
) {
    _ownerAddress = ownerAddress;
    _gameServerAddress = gameServerAddress;
@->    _fighterFarmInstance = FighterFarm(fighterFarmAddress);
@->    _voltageManagerInstance = VoltageManager(voltageManagerAddress);
    isAdmin[_ownerAddress] = true;
    rankedNrnDistribution[0] = 5000 * 10**18;
}
```

The `_fighterFarmInstance` and `_voltageManagerInstance` are initialized only in the constructor and are not modified after that. So they can be made immutable to save gas.

## 13. No need to save both contract address and its instance in storage.

In the `RankedBattle::setStakeAtRiskAddress` function ( https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L192 ):

```solidity
/// @notice Sets the Stake at Risk contract address and instantiates the contract.
/// @dev Only the owner address is authorized to call this function.
/// @param stakeAtRiskAddress The address of the Stake At Risk contract.
function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
    require(msg.sender == _ownerAddress);
@->    _stakeAtRiskAddress = stakeAtRiskAddress;
@->    _stakeAtRiskInstance = StakeAtRisk(_stakeAtRiskAddress);
}
```

There's no need to store both `_stakeAtRiskAddress` and `_stakeAtRiskInstance` in storage. As generally in this contract, only instances are stored for other contracts, it would be better to remove the `_stakeAtRiskAddress` from storage and it can be accessed using `address(_stakeAtRiskInstance)` whenever required.
