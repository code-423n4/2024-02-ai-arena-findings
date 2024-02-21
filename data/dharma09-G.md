## Ai-Arena GAS OPTIMIZATIONS
## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime.

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations.

# Gas report

## Table Of Contents


### [G-01] Move lesser gas costing require checks to the top

**INSTANCE - 1 Details** 
 external call to another contract generally consumes more gas compared to a state read within the same contract. This is because making an external call involves interacting with another contract on the blockchain, which incurs additional costs such as message passing and potential execution of code in the external contract.

**Proof of Code**
- [RankedBattle.sol#L248](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L248)
```solidity
File: src/RankedBattle.sol
244:     function stakeNRN(uint256 amount, uint256 tokenId) external { 
        require(amount > 0, "Amount cannot be 0");
        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
        require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance");
        require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round"); //@audit check first bcz above line check with external call

        _neuronInstance.approveStaker(msg.sender, address(this), amount);
        bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
        if (success) {
            if (amountStaked[tokenId] == 0) { 
                _fighterFarmInstance.updateFighterStaking(tokenId, true);
            }
            amountStaked[tokenId] += amount; //@here
            globalStakedAmount += amount;
            stakingFactor[tokenId] = _getStakingFactor( 
                tokenId, 
                _stakeAtRiskInstance.getStakeAtRisk(tokenId)
            );
            _calculatedStakingFactor[tokenId][roundId] = true;
            emit Staked(msg.sender, amount);
        }
    }
```
**Optimized code:**
```diff
diff --git a/src/RankedBattle.sol b/src/RankedBattle.sol
index 53a89ca..67dd4f0 100644
--- a/src/RankedBattle.sol
+++ b/src/RankedBattle.sol
@@ -241,21 +241,21 @@ contract RankedBattle {
     /// @notice Stakes NRN tokens.
     /// @param amount The amount of NRN tokens to stake.
     /// @param tokenId The ID of the fighter to stake.
     function stakeNRN(uint256 amount, uint256 tokenId) external { 
         require(amount > 0, "Amount cannot be 0");
+        require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");
         require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
         require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance");
-        require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");
 
         _neuronInstance.approveStaker(msg.sender, address(this), amount);
         bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
         if (success) {
            if (amountStaked[tokenId] == 0) {
                 _fighterFarmInstance.updateFighterStaking(tokenId, true);
             }
            amountStaked[tokenId] += amount;
             globalStakedAmount += amount;
            stakingFactor[tokenId] = _getStakingFactor(
                 tokenId, 
                 _stakeAtRiskInstance.getStakeAtRisk(tokenId)
             );
```
**INSTANCE - 2 Details** 

check ` require(hasRole(MINTER_ROLE, msg.sender) `first bcz other check use function call which is more consume more gas
**Proof of Code:**
- [Neuron.sol#L155](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L155)
```solidity
File: src/Neuron.sol
155:    function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint"); //@audit check first bcz above check use external function which is more consume more gas
        _mint(to, amount);
    }
```
**Optimized code:**

```diff
diff --git a/src/Neuron.sol b/src/Neuron.sol
index d400a49..19adbec 100644
--- a/src/Neuron.sol
+++ b/src/Neuron.sol
@@ -153,8 +153,9 @@ contract Neuron is ERC20, AccessControl {
     /// @param to The address to which the tokens will be minted.
     /// @param amount The amount of tokens to be minted.
     function mint(address to, uint256 amount) public virtual {
-        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
         require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
+        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");     
         _mint(to, amount);
     }
```

**Instance 3 Details**
check `require(mergingPortion <= 100);` first because its compare numeric value which is chaeper to revert first.
**Proof of Code**
- [RankedBattle.sol#L331C4-L335C26](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L331C4-L335C26)
```solidity
File: src/RankedBattle.sol
322:   function updateBattleRecord(
        uint256 tokenId, 
        uint256 mergingPortion,
        uint8 battleResult,
        uint256 eloFactor,
        bool initiatorBool
    ) 
        external 
    {   
        require(msg.sender == _gameServerAddress);
        require(mergingPortion <= 100); //@audit check first
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
**Optimized code:**

```diff
diff --git a/src/RankedBattle.sol b/src/RankedBattle.sol
index 53a89ca..33146ea 100644
--- a/src/RankedBattle.sol
+++ b/src/RankedBattle.sol
@@ -328,8 +328,8 @@ contract RankedBattle {
     ) 
         external 
     {   
-        require(msg.sender == _gameServerAddress);
         require(mergingPortion <= 100);
+        require(msg.sender == _gameServerAddress);
         address fighterOwner = _fighterFarmInstance.ownerOf(tokenId);
         require(
             !initiatorBool ||
```
### [G-02] Cache redundant value  to avoid re-calculate value on each iteration
Implementing this change would help save at least 100 gas units per iteration of the loop. The diff below shows how the code should be refactored:

**Details**

**Proof of Code**
- [FighterFarm.sol#L245C4-L261C10](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L245C4-L261C10)
```solidity
File: src/FighterFarm.sol
233: function redeemMintPass(
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
            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i])); //@audit cache mintpassIdsToBurn[i]
            _mintpassInstance.burn(mintpassIdsToBurn[i]);
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
**Optimized code:**

```diff
diff --git a/src/FighterFarm.sol b/src/FighterFarm.sol
index 06ee3e6..61aaaca 100644
--- a/src/FighterFarm.sol
+++ b/src/FighterFarm.sol
@@ -247,8 +247,9 @@ contract FighterFarm is ERC721, ERC721Enumerable {
             modelHashes.length == modelTypes.length
         );
         for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
-            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
-            _mintpassInstance.burn(mintpassIdsToBurn[i]);
+            uint256 mintpassIdsToBurn = mintpassIdsToBurn[i];
+            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn));
+            _mintpassInstance.burn(mintpassIdsToBurn);
             _createNewFighter(
                 msg.sender, 
                 uint256(keccak256(abi.encode(mintPassDnas[i]))), 
```
### [G-03] Avoid Redundant Storage Reads: 
Cache the result of attributeToDnaDivisor[attributes[i]] to avoid redundant storage reads.

**INSTANCE - 1 Details**

**Proof of Code**
- [AiArenaHelper.sol#L105C10-L110C18](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L105C10-L110C18)
```solidity
File: src/AiArenaHelper.sol
83:  function createPhysicalAttributes(
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
                    uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100; //@audit cache attributeToDnaDivisor[attributes[i]];   
                    uint256 attributeIndex = dnaToIndex(generation, rarityRank, attributes[i]);
                    finalAttributeProbabilityIndexes[i] = attributeIndex;
```

**Optimized code:**
```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..252f8bf 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -104,7 +104,8 @@ contract AiArenaHelper {
                 ) {
                     finalAttributeProbabilityIndexes[i] = 50;
                 } else {
-                    uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;
+                    uint256 divisor = attributeToDnaDivisor[attributes[i]]; 
+                    uint256 rarityRank = (dna / divisor) % 100;  
                     uint256 attributeIndex = dnaToIndex(generation, rarityRank, attributes[i]);
                     finalAttributeProbabilityIndexes[i] = attributeIndex;
                 }
```
**INSTANCE - 2 Details**
This code reduces the number of SSTORE operations by storing the attributeProbabilities[generation] in a local variable and updating it in memory.

**Proof of Code**
- [AiArenaHelper.sol#L144C4-L151C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L144C4-L151C6)
```solidity
 File: src/AiArenaHelper.sol
 145: function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i]; //@audit cache attributeProbabilities[generation] in local variable than read
        }
    }

```
**Optimized code:**
```diff
diff --git a/src/MergingPool.sol b/src/MergingPool.sol
index 8efc856..585dfbe 100644
--- a/src/MergingPool.sol
+++ b/src/MergingPool.sol
@@ -115,16 +115,17 @@ contract MergingPool {
     /// @dev The function will update the winnerAddresses mapping with the addresses of the winners.
     /// @dev The function will also reset the fighterPoints of the winners to zero.
     /// @param winners The array of token IDs representing the winners.
-    function pickWinner(uint256[] calldata winners) external {
+    function pickWinner(uint256[] calldata winners) external { 
         require(isAdmin[msg.sender]);
         require(winners.length == winnersPerPeriod, "Incorrect number of winners");
         require(!isSelectionComplete[roundId], "Winners are already selected");
         uint256 winnersLength = winners.length;
         address[] memory currentWinnerAddresses = new address[](winnersLength);
         for (uint256 i = 0; i < winnersLength; i++) {
-            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
-            totalPoints -= fighterPoints[winners[i]];
-            fighterPoints[winners[i]] = 0;
+            uint256 winner = winners[i];
+            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winner); 
+            totalPoints -= fighterPoints[winner];
+            fighterPoints[winner] = 0;
         }
         winnerAddresses[roundId] = currentWinnerAddresses;
         isSelectionComplete[roundId] = true;
```
### [G-04] Use Local Variable for `_stakeAtRiskAddress`
**Details**
 Instead of accessing _stakeAtRiskAddress multiple times directly, use a local variable to potentially save gas.
**Proof of Code**
- [RankedBattle.sol#L192C3-L197C1](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L192C3-L197C1)
```solidity
File: src/RankedBattle.sol
192: function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
        require(msg.sender == _ownerAddress);
        _stakeAtRiskAddress = stakeAtRiskAddress;
        _stakeAtRiskInstance = StakeAtRisk(_stakeAtRiskAddress);//@audit use local variable
    }
```
**Optimized code:**
This change ensures that _stakeAtRiskAddress is accessed through a local variable, which can save small gas.

```diff
192: function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
        require(msg.sender == _ownerAddress);
        _stakeAtRiskAddress = stakeAtRiskAddress;
-        _stakeAtRiskInstance = StakeAtRisk(_stakeAtRiskAddress);
+        _stakeAtRiskInstance = StakeAtRisk(stakeAtRiskAddress);
    }
```

### [G-05] Directly modifies the value in the storage location without reading the current value first

**INSTANCE - 1 Details:**
using pre-increment `(++i)` instead of the addition assignment `(i += 1)` can sometimes result in more efficient code. By doing this we can avoid state variable read as a result we can save gas.

**Proof of Code:**
- [GameItems.sol#L230C2-L235C6](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L230C2-L235C6)
```solidity
File: src/GameItems.sol
230:        if (!transferable) {
          emit Locked(_itemCount);  
        }
        setTokenURI(_itemCount, tokenURI);
        _itemCount += 1; //@audit
```
The idea here is that using `++i` directly modifies the value in the storage location without reading the current value first, potentially saving gas compared to `i += 1`, which involves reading the current value, adding 1, and then writing it back.

**Optimized code:**
```diff
230:        if (!transferable) {
          emit Locked(_itemCount);
        }
        setTokenURI(_itemCount, tokenURI);
-        _itemCount += 1;
+        ++ _itemCount;
```
>Before : | createGameItem                       | 3543            | 170988 | 178091 | 178091 | 36      |
>After  : | createGameItem                       | 3543            | 
170975 | 178077 | 178077 | 36      |

**INSTANCE - 2**

**Proof of Code**
- [FighterFarm.sol#L128C5-L135C1](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L128C5-L135C1)
```solidity
File: src/FighterFarm.sol
129:  function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
        generation[fighterType] += 1; //@audit
        maxRerollsAllowed[fighterType] += 1;
        return generation[fighterType];
    }

```
**Optimized code:**

```diff
diff --git a/src/FighterFarm.sol b/src/FighterFarm.sol
index 06ee3e6..c8a82d9 100644
--- a/src/FighterFarm.sol
+++ b/src/FighterFarm.sol
@@ -128,8 +128,8 @@ contract FighterFarm is ERC721, ERC721Enumerable {
     /// @return Generation count of the fighter type.
     function incrementGeneration(uint8 fighterType) external returns (uint8) {
         require(msg.sender == _ownerAddress);
-        generation[fighterType] += 1;
-        maxRerollsAllowed[fighterType] += 1;
+        ++generation[fighterType]; 
+        ++maxRerollsAllowed[fighterType];
```
>| src/FighterFarm.sol:FighterFarm contract |                 |        |        |        |         |
>|------------------------------------------|-----------------|--------|--------|--------|---------|
>| Function Name                            | min             | avg    | median | max    | # calls |
>| incrementGeneration                      | 2567            | 16251  | 15751  | 30935  | 4       |
>| incrementGeneration                      | 2567            | 16226  | 15726  | 30885  | 4       |

**INSTANCE - 3**

**Proof of Code**
- [FighterFarm.sol#L283C2-L296C1](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L283C2-L296C1)
```solidity
File: src/FighterFarm.sol
283:  function updateModel(
        uint256 tokenId, 
        string calldata modelHash,
        string calldata modelType
    ) 
        external
    {
        require(msg.sender == ownerOf(tokenId));
        fighters[tokenId].modelHash = modelHash;
        fighters[tokenId].modelType = modelType;
        numTrained[tokenId] += 1; //@audit
        totalNumTrained += 1;
    }
```
**Optimized code:**

```diff
diff --git a/src/FighterFarm.sol b/src/FighterFarm.sol
index 06ee3e6..4e8711d 100644
--- a/src/FighterFarm.sol
+++ b/src/FighterFarm.sol
@@ -290,8 +290,8 @@ contract FighterFarm is ERC721, ERC721Enumerable {
         require(msg.sender == ownerOf(tokenId));
         fighters[tokenId].modelHash = modelHash;
         fighters[tokenId].modelType = modelType;
-        numTrained[tokenId] += 1;
-        totalNumTrained += 1;
+        ++numTrained[tokenId];
+        ++totalNumTrained;
     }
```
>Before : | updateModel                              | 1149            | 17094  | 17094  | 33040  | 2       |
>After  : | updateModel                              | 1149            | 17071  | 17071  | 32993  | 2       |

**INSTANCE - 4**

**Proof of Code**
- [FighterFarm.sol#L371](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L371)
```solidity
File: src/FighterFarm.sol
371: function reRoll(uint8 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

        _neuronInstance.approveSpender(msg.sender, rerollCost);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
        if (success) {
            numRerolls[tokenId] += 1; //@audit
            uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
            (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
            fighters[tokenId].element = element;
            fighters[tokenId].weight = weight;
            fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
                newDna,
                generation[fighterType],
                fighters[tokenId].iconsType,
                fighters[tokenId].dendroidBool
            );
            _tokenURIs[tokenId] = "";
        }
    }    
```
**Optimized code:**

```diff
function reRoll(uint8 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

        _neuronInstance.approveSpender(msg.sender, rerollCost);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
        if (success) {
-            numRerolls[tokenId] += 1;
+            ++numRerolls[tokenId];
            uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
            (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);    
```
**INSTANCE - 5**

**Proof of Code**
- [MergingPool.sol#L150](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L150)
```diff
File: src/MergingPool.sol
138:  uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
-            numRoundsClaimed[msg.sender] += 1;//@audit
+            ++numRoundsClaimed[msg.sender];
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
-                    claimIndex += 1;
+                    ++claimIndex;
                }
            }
```
**INSTANCE - 6**

**Proof of Code**
- [MergingPool.sol#L131](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L131)
```diff
File: src/MergingPool.sol
118: function pickWinner(uint256[] calldata winners) external {
        require(isAdmin[msg.sender]);
        require(winners.length == winnersPerPeriod, "Incorrect number of winners");
        require(!isSelectionComplete[roundId], "Winners are already selected");
        uint256 winnersLength = winners.length;
        address[] memory currentWinnerAddresses = new address[](winnersLength);
        for (uint256 i = 0; i < winnersLength; i++) {
            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]); 
            totalPoints -= fighterPoints[winners[i]];
            fighterPoints[winners[i]] = 0;
        }
        winnerAddresses[roundId] = currentWinnerAddresses;
        isSelectionComplete[roundId] = true;
-        roundId += 1; //@audit
+        ++roundId;
    }
```
**INSTANCE - 7**

**Proof of Code**
- [RankedBattle.sol#L348](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L348)
```diff
File: src/RankedBattle.sol
348:  }
        if (initiatorBool) {
            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
        }
-        totalBattles += 1; //@audit
+        ++totalBattles;
```
**INSTANCE - 8**

**Proof of Code**
- [RankedBattle.sol#L304](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L304)
```diff
File: src/RankedBattle.sol
305:  for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound];
-            numRoundsClaimed[msg.sender] += 1; //@audit
+            ++numRoundsClaimed[msg.sender];
        }
```
**INSTANCE - 9**

**Proof of Code**
- [FighterFarm.sol#L378](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L378)
```diff
File: src/FighterFarm.sol
377:         if (success) {
-            numRerolls[tokenId] += 1; //@audit
+            ++numRerolls[tokenId];
            uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
            (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
```
### [G-06] Do not initialized state variable

**Details**
Instead of initialized state varibable we can use variabl default value.For uint256 Default value is 0.
**Proof of Code**
- [GameItems.sol#L64](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L64)
```diff
File: src/GameItems.sol
-64:     uint256 _itemCount = 0;
+64:     uint256 _itemCount;
```
>| Deployment Cost                      | Deployment Size |        |        |        |         |
>Before : | 2157844                              | 10818           |        |        |        |         |
>After  : | 2155638                              | 10813           |        |        |        |         |
>save 2206 Deployment gas

**INSTANCE - 2**

**Proof of Code**
- [MergingPool.sol#L32](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L32)
```diff
File: src/MergingPool.sol
-32:  uint256 public totalPoints = 0;
+32:  uint256 public totalPoints;
```
| Deployment Cost                          | Deployment Size |        |        |        |         |
>Before | 912202                                   | 4340            |        |        |        |         |
>After  | 909996                                   | 4335            |        |        |        |         |
>save 2206 Deployment gas

**INSTANCE - 3**

**Proof of Code**
- [RankedBattle.sol#L56C3-L59C43](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L56C3-L59C43)
```diff
File: src/RankedBattle.sol
55:    /// @notice Number of total battles.
56:    uint256 public totalBattles = 0; //@audit state varible intialize with zero value

58:    /// @notice Number of overall staked amount.
59:    uint256 public globalStakedAmount = 0; //@audit
```
>save 4412 Deployment gas