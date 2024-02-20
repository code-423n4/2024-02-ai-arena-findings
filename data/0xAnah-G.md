# AI ARENA GAS OPTIMIZATIONS


## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."


## [G-01] Same conditional in loop iterations
Some conditionals in the loop do not depend on the loop's iterations. Having to evaluate this condition for every iteration of the loop is gas consuming and redundant rather the conditional should be evaluated before the loop, the result saved to a variable and the variable be used in the loop.

### 1 Instance
1. #### `iconsType == 2`, `iconsType > 0` and `iconsType == 3` conditionals do not depend on the loop iterations
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L110-#L104

In the `AiArenaHelper.createPhysicalAttributes()` function as shown below the conditions `iconsType == 2`, `iconsType > 0` and `iconsType == 3` were checked for every iteration of the loop but these conditionals do not depend on the loop iterations. We can make the function more gas efficient by evaluating these conditionals before the loop then save the result to their respective variable then these variables be used in the loop. The diff below shows how the code could be refactored:

```solidity
file: src/AiArenaHelper.sol

83:     function createPhysicalAttributes(
84:         uint256 dna, 
85:         uint8 generation, 
86:         uint8 iconsType, 
87:         bool dendroidBool
88:     ) 
89:         external 
90:         view 
91:         returns (FighterOps.FighterPhysicalAttributes memory) 
92:     {
93:         if (dendroidBool) {
94:             return FighterOps.FighterPhysicalAttributes(99, 99, 99, 99, 99, 99);
95:         } else {
96:             uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);
97: 
98:             uint256 attributesLength = attributes.length;
99:             for (uint8 i = 0; i < attributesLength; i++) {
100:                if (
101:                  i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
102:                  i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
103:                  i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
104:                ) {
105:                    finalAttributeProbabilityIndexes[i] = 50;
106:                } else {
107:                    uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;
108:                    uint256 attributeIndex = dnaToIndex(generation, rarityRank, attributes[i]);
109:                    finalAttributeProbabilityIndexes[i] = attributeIndex;
110:                }
111:            }
112:            return FighterOps.FighterPhysicalAttributes(
113:                finalAttributeProbabilityIndexes[0],
114:                finalAttributeProbabilityIndexes[1],
115:                finalAttributeProbabilityIndexes[2],
116:                finalAttributeProbabilityIndexes[3],
117:                finalAttributeProbabilityIndexes[4],
118:                finalAttributeProbabilityIndexes[5]
119:            );
120:        }
121:    }
```

```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..67cc7e8 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -96,11 +96,15 @@ contract AiArenaHelper {
             uint256[] memory finalAttributeProbabilityIndexes = new uint[](attributes.length);

             uint256 attributesLength = attributes.length;
+            bool isBeteHelmet = iconsType == 2;
+            bool isRedDiamond = iconsType > 0;
+            bool isBowlingBall = iconsType == 3;
+
             for (uint8 i = 0; i < attributesLength; i++) {
                 if (
-                  i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
-                  i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
-                  i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
+                  i == 0 && isBeteHelmet || // Custom icons head (beta helmet)
+                  i == 1 && isRedDiamond || // Custom icons eyes (red diamond)
+                  i == 4 && isBowlingBall // Custom icons hands (bowling ball)
                 ) {
                     finalAttributeProbabilityIndexes[i] = 50;
                 } else {
```



## [G-02] Redundant state variable getters
Getters for public state variables are automatically generated so there is no need to code
them manually and lose gas.

### Instances
1. #### Make `attributeProbabilities` mapping variable `private` or `internal` since a getter function was defined for it.
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L30

The solidity compiler would automatically create a getter function for the `attributeProbabilities` mapping since it is declared as a `public` variable but a getter function `getAttributeProbabilities()` was also declared in the contract for the same variable thereby making it two getter functions for the same variable in the contract. We could rectify this issue by making the `mapUnits` variable private or internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: src/AiArenaHelper.sol

30:    mapping(uint256 => mapping(string => uint8[])) public attributeProbabilities;    //@audit make internal or private
.
.
.
157:    function getAttributeProbabilities(uint256 generation, string memory attribute) 
158:        public 
159:        view 
160:        returns (uint8[] memory) 
161:    {
162:        return attributeProbabilities[generation][attribute];
163:    }    
```

```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..7181ccf 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -27,7 +27,7 @@ contract AiArenaHelper {
     //////////////////////////////////////////////////////////////*/

     /// @notice Mapping tracking fighter generation to attribute probabilities
-    mapping(uint256 => mapping(string => uint8[])) public attributeProbabilities;
+    mapping(uint256 => mapping(string => uint8[])) internal attributeProbabilities;

     /// @notice Mapping of attribute to DNA divisors
     mapping(string => uint8) public attributeToDnaDivisor;
```

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L122




## [G-03] Add unchecked blocks for subtractions where the operands cannot underflow
The solidity compiler introduces some checks to avoid an underflow, but in some scenarios where it is impossible for underflow to occur we can use unchecked blocks to have some gas savings.

### 2 Instances
1. #### The calculation `amountStaked[tokenId] -= amount` can be unchecked
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L275

The if statement `if (amount > amountStaked[tokenId]) {amount = amountStaked[tokenId]}` ensures that its impossible for the value of `amount` to be greater than the value of `amountStaked[tokenId]` therefore the calulation `amountStaked[tokenId] -= amount` cannot underflow. Since it cannot underflow we can safely do the calculation in an unchecked block and achieve some gas saving. The diff beolw shows how the code could be refactored: 

```solidity
file: src/RankedBattle.sol

270:    function unstakeNRN(uint256 amount, uint256 tokenId) external {
271:        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
272:        if (amount > amountStaked[tokenId]) {
273:            amount = amountStaked[tokenId];
274:        }
275:        amountStaked[tokenId] -= amount;
276:        globalStakedAmount -= amount;
277:        stakingFactor[tokenId] = _getStakingFactor(
278:            tokenId, 
279:            _stakeAtRiskInstance.getStakeAtRisk(tokenId)
280:        );
281:        _calculatedStakingFactor[tokenId][roundId] = true;
282:        hasUnstaked[tokenId][roundId] = true;
283:        bool success = _neuronInstance.transfer(msg.sender, amount);
284:        if (success) {
285:            if (amountStaked[tokenId] == 0) {
286:                _fighterFarmInstance.updateFighterStaking(tokenId, false);
287:            }
288:            emit Unstaked(msg.sender, amount);
289:        }
290:    }
```

```diff
diff --git a/src/RankedBattle.sol b/src/RankedBattle.sol
index 53a89ca..9b96012 100644
--- a/src/RankedBattle.sol
+++ b/src/RankedBattle.sol
@@ -272,7 +272,9 @@ contract RankedBattle {
         if (amount > amountStaked[tokenId]) {
             amount = amountStaked[tokenId];
         }
-        amountStaked[tokenId] -= amount;
+        unchecked {
+            amountStaked[tokenId] -= amount;
+        }
         globalStakedAmount -= amount;
         stakingFactor[tokenId] = _getStakingFactor(
             tokenId,
```

2. #### The calculation `stakeAtRisk[roundId][fighterId] -= nrnToReclaim` can be unchecked
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L102

The require statement `require(stakeAtRisk[roundId][fighterId] >= nrnToReclaim, "Fighter does not have enough stake at risk")` ensures that the value of `stakeAtRisk[roundId][fighterId]` is greater than or equal to the value of `nrnToReclaim` therefore the calulation `stakeAtRisk[roundId][fighterId] -= nrnToReclaim` cannot underflow. Since it cannot underflow we can safely do the calculation in an unchecked block and achieve some gas saving. The diff beolw shows how the code could be refactored:

```solidity
file: src/StakeAtRisk.sol

93:     function reclaimNRN(uint256 nrnToReclaim, uint256 fighterId, address fighterOwner) external {
94:         require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");
95:         require(
96:             stakeAtRisk[roundId][fighterId] >= nrnToReclaim, 
97:             "Fighter does not have enough stake at risk"
98:         );
99: 
100:        bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
101:        if (success) {
102:            stakeAtRisk[roundId][fighterId] -= nrnToReclaim;
103:            totalStakeAtRisk[roundId] -= nrnToReclaim;
104:            amountLost[fighterOwner] -= nrnToReclaim;
105:            emit ReclaimedStake(fighterId, nrnToReclaim);
106:        }
107:    }
```

```diff
diff --git a/src/StakeAtRisk.sol b/src/StakeAtRisk.sol
index bdc87b5..c10d755 100644
--- a/src/StakeAtRisk.sol
+++ b/src/StakeAtRisk.sol
@@ -99,7 +99,9 @@ contract StakeAtRisk {

         bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
         if (success) {
-            stakeAtRisk[roundId][fighterId] -= nrnToReclaim;
+            unchecked {
+                stakeAtRisk[roundId][fighterId] -= nrnToReclaim;
+            }
             totalStakeAtRisk[roundId] -= nrnToReclaim;
             amountLost[fighterOwner] -= nrnToReclaim;
             emit ReclaimedStake(fighterId, nrnToReclaim);
```


## [G-04] Access mappings directly rather than using accessor functions
Rather than having to read mapping using accessor function you should read the mapping directly as this saves having to do two JUMP instructions, along with stack setup.

### 3 Instances
1. #### Refactor `AiArenaHelper.dnaToIndex()` function to read `attributeProbabilities` mapping directly.
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L174

We can make `AiArenaHelper.dnaToIndex()` function more gas efficient if the `attributeProbabilities[generation][attribute]` mapping is read directly as opposed to reading its value using the `getAttributeProbabilities()` function. Implemeting this would help avoid the gas cost for 2 JUMP opcodes and stack set up for the function invocation. Also having to read the `attributeProbabilities[generation][attribute]` mapping directly would help avoid coping the returned `uint8` array from storage to memory. The diff below shows how the code should be refactored: 

```solidity
file: src/AiArenaHelper.sol

169:    function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
170:        public 
171:        view 
172:        returns (uint256 attributeProbabilityIndex) 
173:    {
174:        uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute); //@audit read mapping directly
175:        
176:        uint256 cumProb = 0;
177:        uint256 attrProbabilitiesLength = attrProbabilities.length;
178:        for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
179:            cumProb += attrProbabilities[i];
180:            if (cumProb >= rarityRank) {
181:                attributeProbabilityIndex = i + 1;
182:                break;
183:            }
184:        }
185:        return attributeProbabilityIndex;
186:    }
```

```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..04ad843 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -171,7 +171,7 @@ contract AiArenaHelper {
         view
         returns (uint256 attributeProbabilityIndex)
     {
-        uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);
+        uint8[] storage attrProbabilities = attributeProbabilities[generation][attribute];

         uint256 cumProb = 0;
         uint256 attrProbabilitiesLength = attrProbabilities.length;
```


2. #### Refactor `RankedBattle.claimNRN()` function to read `rankedNrnDistribution` mapping directly.
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L300

We can make `RankedBattle.claimNRN()` function more gas efficient if the `rankedNrnDistribution[currentRound]` mapping is read directly as opposed to reading its value using the `getNrnDistribution(currentRound)` function. Implementing this would help avoid the gas cost for 2 JUMP opcodes and stack set up for the function invocation. Also considering this is done in a loop would even reduce the gas cost of the `RankedBattle.claimNRN()` function astronomically. The diff below shows how the code could be refactored: 

```solidity
file: src/RankedBattle.sol

294:    function claimNRN() external {
295:        require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
296:        uint256 claimableNRN = 0;
297:        uint256 nrnDistribution;
298:        uint32 lowerBound = numRoundsClaimed[msg.sender];
299:        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
300:            nrnDistribution = getNrnDistribution(currentRound); //@audit read mapping directly
301:            claimableNRN += (
302:                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
303:            ) / totalAccumulatedPoints[currentRound];
304:            numRoundsClaimed[msg.sender] += 1;
305:        }
306:        if (claimableNRN > 0) {
307:            amountClaimed[msg.sender] += claimableNRN;
308:            _neuronInstance.mint(msg.sender, claimableNRN);
309:            emit Claimed(msg.sender, claimableNRN);
310:        }
311:    }
```

```diff
diff --git a/src/RankedBattle.sol b/src/RankedBattle.sol
index 53a89ca..c1a0f2c 100644
--- a/src/RankedBattle.sol
+++ b/src/RankedBattle.sol
@@ -297,7 +297,7 @@ contract RankedBattle {
         uint256 nrnDistribution;
         uint32 lowerBound = numRoundsClaimed[msg.sender];
         for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
-            nrnDistribution = getNrnDistribution(currentRound);
+            nrnDistribution = rankedNrnDistribution[currentRound];
             claimableNRN += (
                 accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution
             ) / totalAccumulatedPoints[currentRound];
```


3. #### Refactor `RankedBattle.getUnclaimedNRN()` function to read `rankedNrnDistribution` mapping directly.
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L391

We can make `RankedBattle.getUnclaimedNRN()` function more gas efficient if the `rankedNrnDistribution` mapping is read directly as opposed to reading its value using the `getNrnDistribution(currentRound)` function. Implementing this would help avoid the gas cost for 2 JUMP opcodes and stack set up for the function invocation. Also considering this is done in a loop would even reduce the gas cost of the `RankedBattle.getUnclaimedNRN()` function astronomically. The diff below shows how the code could be refactored: 

```solidity
file: src/RankedBattle.sol

386:    function getUnclaimedNRN(address claimer) public view returns(uint256) {
387:        uint256 claimableNRN = 0;
388:        uint256 nrnDistribution;   
389:        uint32 lowerBound = numRoundsClaimed[claimer];
390:        for (uint32 i = lowerBound; i < roundId; i++) {
391:            nrnDistribution = getNrnDistribution(i);
392:            claimableNRN += (
393:                accumulatedPointsPerAddress[claimer][i] * nrnDistribution
394:            ) / totalAccumulatedPoints[i];
395:        }
396:        return claimableNRN;
397:    } 
```

```diff
diff --git a/src/RankedBattle.sol b/src/RankedBattle.sol
index 53a89ca..bc4d36e 100644
--- a/src/RankedBattle.sol
+++ b/src/RankedBattle.sol
@@ -388,7 +388,7 @@ contract RankedBattle {
         uint256 nrnDistribution;
         uint32 lowerBound = numRoundsClaimed[claimer];
         for (uint32 i = lowerBound; i < roundId; i++) {
-            nrnDistribution = getNrnDistribution(i);
+            nrnDistribution = rankedNrnDistribution[i];
             claimableNRN += (
                 accumulatedPointsPerAddress[claimer][i] * nrnDistribution
             ) / totalAccumulatedPoints[i];
```


## [G-05] Using `delete` to clear mappings is cheaper than assigning null memory values.

### 2 Instances
1. #### use `delete` to clear `attributeProbabilities` mapping
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L149

In the `AiArenaHelper.deleteAttributeProbabilities()` function as shown below the `attributeProbabilities` mapping was cleared by assigning it empty memory `uint8` arrays which isn't gas efficient and also considering that this was done in a loop would make the function very gas in-efficient. We can rectify this by using the `delete` operator to clear the mapping as it is cheaper. The diff below shows how the code should be refactored: 

```solidity
file: src/AiArenaHelper.sol

144:    function deleteAttributeProbabilities(uint8 generation) public {
145:        require(msg.sender == _ownerAddress);
146:
147:        uint256 attributesLength = attributes.length;
148:        for (uint8 i = 0; i < attributesLength; i++) {
149:            attributeProbabilities[generation][attributes[i]] = new uint8[](0); //@audit use delete instead
150:        }
151:    }
```

```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..7ee70fd 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -146,7 +146,7 @@ contract AiArenaHelper {

         uint256 attributesLength = attributes.length;
         for (uint8 i = 0; i < attributesLength; i++) {
-            attributeProbabilities[generation][attributes[i]] = new uint8[](0);
+            delete attributeProbabilities[generation][attributes[i]];
         }
     }
```


2. #### use `delete` to clear `_tokenURIs` mapping
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L389

In the `FighterFarm.reRoll()` function as shown below the `_tokenURIs` mapping was cleared by assigning it an empty string which isn't gas efficient. We can rectify this by using the `delete` operator to clear the mapping as it is cheaper. The diff below shows how the code should be refactored: 

```solidity
file: src/FighterFarm.sol

370:    function reRoll(uint8 tokenId, uint8 fighterType) public {
371:        require(msg.sender == ownerOf(tokenId));
372:        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
373:        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");
374:
375:        _neuronInstance.approveSpender(msg.sender, rerollCost);
376:        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
377:        if (success) {
378:            numRerolls[tokenId] += 1;
379:            uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
380:            (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
381:            fighters[tokenId].element = element;
382:            fighters[tokenId].weight = weight;
383:            fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
384:                newDna,
385:                generation[fighterType],
386:                fighters[tokenId].iconsType,
387:                fighters[tokenId].dendroidBool
388:            );
389:            _tokenURIs[tokenId] = "";   //@audit use delete instead
390:        }
391:    } 
```

```diff
diff --git a/src/FighterFarm.sol b/src/FighterFarm.sol
index 06ee3e6..6495892 100644
--- a/src/FighterFarm.sol
+++ b/src/FighterFarm.sol
@@ -386,7 +386,7 @@ contract FighterFarm is ERC721, ERC721Enumerable {
                 fighters[tokenId].iconsType,
                 fighters[tokenId].dendroidBool
             );
-            _tokenURIs[tokenId] = "";
+            delete _tokenURIs[tokenId];
         }
     }
```


## [G-06] State variable read and written in a loop
The code should be refactored such that reads and updates made to the state variable are instead accumulated/tracked in a local variable, then be written a single time outside the loop, converting a Gsreset (2900 gas) to a stack write for each iteration

#### Please note this instance is not included in the bots report.

### Instances
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L304
```solidity
file: src/RankedBattle.sol

294:    function claimNRN() external {
295:        require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
296:        uint256 claimableNRN = 0;
297:        uint256 nrnDistribution;
298:        uint32 lowerBound = numRoundsClaimed[msg.sender];
299:        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
300:            nrnDistribution = getNrnDistribution(currentRound);
301:            claimableNRN += (
302:                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
303:            ) / totalAccumulatedPoints[currentRound];
304:            numRoundsClaimed[msg.sender] += 1;
305:        }
306:        if (claimableNRN > 0) {
307:            amountClaimed[msg.sender] += claimableNRN;
308:            _neuronInstance.mint(msg.sender, claimableNRN);
309:            emit Claimed(msg.sender, claimableNRN);
310:        }
311:    }
```

```diff
diff --git a/src/RankedBattle.sol b/src/RankedBattle.sol
index 53a89ca..b414fb0 100644
--- a/src/RankedBattle.sol
+++ b/src/RankedBattle.sol
@@ -301,8 +301,9 @@ contract RankedBattle {
             claimableNRN += (
                 accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution
             ) / totalAccumulatedPoints[currentRound];
-            numRoundsClaimed[msg.sender] += 1;
+            lowerBound += 1;
         }
+        numRoundsClaimed[msg.sender] = lowerBound;
         if (claimableNRN > 0) {
             amountClaimed[msg.sender] += claimableNRN;
             _neuronInstance.mint(msg.sender, claimableNRN);
```



## [G-07] Refactor external/internal function to avoid unnecessary External Calls
The functions below perform external calls that are previously performed in the functions that invoke them. We can refactor the external/internal functions to pass the cached external calls into them and avoid the extra external calls that would otherwise take place in the internal functions.

### 1 Instance
1. #### Refactor external function `updateBattleRecord()` and private function `_addResultPoints()` to avoid unnecessary external calls.
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L322-#L349
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L416-#L500

The `updateBattleRecord()` function as shown below performs an external call `_stakeAtRiskInstance.getStakeAtRisk(tokenId)`
and caches the external call returned value. The `updateBattleRecord()` also invokes private function `_addResultPoints()` which performs the same external call (`_stakeAtRiskInstance.getStakeAtRisk(tokenId)`) immediately it's invoked. Doing it this this way is redundant and a waste of gas rather we should refactor the external function `updateBattleRecord()` and the private function `_addResultPoints()` such that the cached external call returned value `stakeAtRisk` is passed as an argument to the private function therefore this eliminates the need to perform the `_stakeAtRiskInstance.getStakeAtRisk(tokenId)` external call in the private function. Also the private function reads the state variable `_stakeAtRiskInstance`
which is also read by the external function so we could also pass state variable also as an argument to the private function.  

```solidity
file: src/RankedBattle.sol

322:    function updateBattleRecord(
323:        uint256 tokenId, 
324:        uint256 mergingPortion,
325:        uint8 battleResult,
326:        uint256 eloFactor,
327:        bool initiatorBool
328:    ) 
329:        external 
330:    {   
331:        require(msg.sender == _gameServerAddress);
332:        require(mergingPortion <= 100);
333:        address fighterOwner = _fighterFarmInstance.ownerOf(tokenId);
334:        require(
335:            !initiatorBool ||
336:            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
337:            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
338:        );
339:
340:        _updateRecord(tokenId, battleResult);
341:        uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
342:        if (amountStaked[tokenId] + stakeAtRisk > 0) {
343:            _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
344:        }
345:        if (initiatorBool) {
346:            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
347:        }
348:        totalBattles += 1;
349:    }
.
.
.
416:    function _addResultPoints(
417:        uint8 battleResult, 
418:        uint256 tokenId, 
419:        uint256 eloFactor, 
420:        uint256 mergingPortion,
421:        address fighterOwner
422:    ) 
423:        private 
424:    {
425:        uint256 stakeAtRisk;
426:        uint256 curStakeAtRisk;
427:        uint256 points = 0;
428:
429:        /// Check how many NRNs the fighter has at risk
430:        stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
431:
432:        /// Calculate the staking factor if it has not already been calculated for this round 
433:        if (_calculatedStakingFactor[tokenId][roundId] == false) {
434:            stakingFactor[tokenId] = _getStakingFactor(tokenId, stakeAtRisk);
435:            _calculatedStakingFactor[tokenId][roundId] = true;
436:        }
437:
438:        /// Potential amount of NRNs to put at risk or retrieve from the stake-at-risk contract
439:        curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
440:        if (battleResult == 0) {
441:            /// If the user won the match
442:
443:            /// If the user has no NRNs at risk, then they can earn points
444:            if (stakeAtRisk == 0) {
445:                points = stakingFactor[tokenId] * eloFactor;
446:            }
447:
448:            /// Divert a portion of the points to the merging pool
449:            uint256 mergingPoints = (points * mergingPortion) / 100;
450:            points -= mergingPoints;
451:            _mergingPoolInstance.addPoints(tokenId, mergingPoints);
452:
453:            /// Do not allow users to reclaim more NRNs than they have at risk
454:            if (curStakeAtRisk > stakeAtRisk) {
455:                curStakeAtRisk = stakeAtRisk;
456:            }
457:
458:            /// If the user has stake-at-risk for their fighter, reclaim a portion
459:            /// Reclaiming stake-at-risk puts the NRN back into their staking pool
460:            if (curStakeAtRisk > 0) {
461:                _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
462:                amountStaked[tokenId] += curStakeAtRisk;
463:            }
464:
465:            /// Add points to the fighter for this round
466:            accumulatedPointsPerFighter[tokenId][roundId] += points;
467:            accumulatedPointsPerAddress[fighterOwner][roundId] += points;
468:            totalAccumulatedPoints[roundId] += points;
469:            if (points > 0) {
470:                emit PointsChanged(tokenId, points, true);
471:            }
472:        } else if (battleResult == 2) {
473:            /// If the user lost the match
474:
475:            /// Do not allow users to lose more NRNs than they have in their staking pool
476:            if (curStakeAtRisk > amountStaked[tokenId]) {
477:                curStakeAtRisk = amountStaked[tokenId];
478:            }
479:            if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
480:                /// If the fighter has a positive point balance for this round, deduct points 
481:                points = stakingFactor[tokenId] * eloFactor;
482:                if (points > accumulatedPointsPerFighter[tokenId][roundId]) {
483:                    points = accumulatedPointsPerFighter[tokenId][roundId];
484:                }
485:                accumulatedPointsPerFighter[tokenId][roundId] -= points;
486:                accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
487:                totalAccumulatedPoints[roundId] -= points;
488:                if (points > 0) {
489:                    emit PointsChanged(tokenId, points, false);
490:                }
491:            } else {
492:                /// If the fighter does not have any points for this round, NRNs become at risk of being lost
493:                bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
494:                if (success) {
495:                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
496:                    amountStaked[tokenId] -= curStakeAtRisk;
497:                }
498:            }
499:        }
500:    }
```

```diff
diff --git a/src/RankedBattle.sol b/src/RankedBattle.sol
index 53a89ca..1a28835 100644
--- a/src/RankedBattle.sol
+++ b/src/RankedBattle.sol
@@ -338,9 +338,10 @@ contract RankedBattle {
         );

         _updateRecord(tokenId, battleResult);
-        uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
+        StakeAtRisk stakeAtRiskInstance = _stakeAtRiskInstance;
+        uint256 stakeAtRisk = stakeAtRiskInstance.getStakeAtRisk(tokenId);
         if (amountStaked[tokenId] + stakeAtRisk > 0) {
-            _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
+            _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner, stakeAtRisk, stakeAtRiskInstance);
         }
         if (initiatorBool) {
             _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
@@ -418,17 +419,15 @@ contract RankedBattle {
         uint256 tokenId,
         uint256 eloFactor,
         uint256 mergingPortion,
-        address fighterOwner
+        address fighterOwner,
+        uint256 stakeAtRisk,
+        StakeAtRisk stakeAtRiskInstance
     )
         private
     {
-        uint256 stakeAtRisk;
         uint256 curStakeAtRisk;
         uint256 points = 0;

-        /// Check how many NRNs the fighter has at risk
-        stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
-
         /// Calculate the staking factor if it has not already been calculated for this round
         if (_calculatedStakingFactor[tokenId][roundId] == false) {
             stakingFactor[tokenId] = _getStakingFactor(tokenId, stakeAtRisk);
@@ -458,7 +457,7 @@ contract RankedBattle {
             /// If the user has stake-at-risk for their fighter, reclaim a portion
             /// Reclaiming stake-at-risk puts the NRN back into their staking pool
             if (curStakeAtRisk > 0) {
-                _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
+                stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
                 amountStaked[tokenId] += curStakeAtRisk;
             }

@@ -492,7 +491,7 @@ contract RankedBattle {
                 /// If the fighter does not have any points for this round, NRNs become at risk of being lost
                 bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
                 if (success) {
-                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
+                    stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                     amountStaked[tokenId] -= curStakeAtRisk;
                 }
```




## [G-08] Caching the length of calldata increases gas cost instead of reducing it

### Proof of Concept
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CacheCallDataLength {
    

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {
        uint256 len = numbers.length;

        for(uint i; i < len; ++i) {
            total += numbers[i];
        }

    }
}

```
```
test for test/CacheCallDataLength.t.sol:CacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1633)
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
contract NoCacheCallDataLength {
    

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {

        for(uint i; i < numbers.length; ++i) {
            total += numbers[i];
        }
        
    }
}
```
```
test for test/NoCacheCallDataLength.t.sol:NoCacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1628)
```

### 2 Instances
1. #### Reduce gas cost of `pickWinner()` function by not caching the length of calldata variable `winners`.
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L124

```solidity
file: src/MergingPool.sol

118:    function pickWinner(uint256[] calldata winners) external {
119:        require(isAdmin[msg.sender]);
120:        require(winners.length == winnersPerPeriod, "Incorrect number of winners");
121:        require(!isSelectionComplete[roundId], "Winners are already selected");
122:        uint256 winnersLength = winners.length;
123:        address[] memory currentWinnerAddresses = new address[](winnersLength);
124:        for (uint256 i = 0; i < winnersLength; i++) {
125:            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
126:            totalPoints -= fighterPoints[winners[i]];
127:            fighterPoints[winners[i]] = 0;
128:        }
129:        winnerAddresses[roundId] = currentWinnerAddresses;
130:        isSelectionComplete[roundId] = true;
131:        roundId += 1;
132:    }
```

```diff
diff --git a/src/MergingPool.sol b/src/MergingPool.sol
index 8efc856..ff84220 100644
--- a/src/MergingPool.sol
+++ b/src/MergingPool.sol
@@ -119,9 +119,8 @@ contract MergingPool {
         require(isAdmin[msg.sender]);
         require(winners.length == winnersPerPeriod, "Incorrect number of winners");
         require(!isSelectionComplete[roundId], "Winners are already selected");
-        uint256 winnersLength = winners.length;
-        address[] memory currentWinnerAddresses = new address[](winnersLength);
-        for (uint256 i = 0; i < winnersLength; i++) {
+        address[] memory currentWinnerAddresses = new address[](winners.length);
+        for (uint256 i = 0; i < winners.length; i++) {
             currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
             totalPoints -= fighterPoints[winners[i]];
             fighterPoints[winners[i]] = 0;
```

2. #### Reduce gas cost of `setupAirdrop()` function by not caching the length of calldata variable `recipients`.
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L131
```solidity
file: src/Neuron.sol

127:    function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
128:        require(isAdmin[msg.sender]);
129:        require(recipients.length == amounts.length);
130:        uint256 recipientsLength = recipients.length;
131:        for (uint32 i = 0; i < recipientsLength; i++) {
132:            _approve(treasuryAddress, recipients[i], amounts[i]);
133:        }
134:    }
```

```diff
diff --git a/src/Neuron.sol b/src/Neuron.sol
index d400a49..7a4a23f 100644
--- a/src/Neuron.sol
+++ b/src/Neuron.sol
@@ -127,8 +127,7 @@ contract Neuron is ERC20, AccessControl {
     function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
         require(isAdmin[msg.sender]);
         require(recipients.length == amounts.length);
-        uint256 recipientsLength = recipients.length;
-        for (uint32 i = 0; i < recipientsLength; i++) {
+        for (uint32 i = 0; i < recipients.length; i++) {
             _approve(treasuryAddress, recipients[i], amounts[i]);
         }
     }
```


## [G-09] Avoid using state variable in emit
In scenarios where you have a memory, calldata variable or parameter with the same value as the state variable you should use the memory, calldata variable or parameter in the emit statement rather than state variable.

### 1 Instance

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L98

The `useVoltageBattery()` function above could be refactored to not use the state variable `ownerVoltage[msg.sender]` in the `VoltageRemaining()` emit statement rather the number `100` should be used instead thereby saving 1 `SLOAD` (warm access) `100` gas units. The refactored code is shown in the diff below:

```solidity
file: src/VoltageManager.sol

93:    function useVoltageBattery() public {
94:        require(ownerVoltage[msg.sender] < 100);
95:        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
96:        _gameItemsContractInstance.burn(msg.sender, 0, 1);
97:        ownerVoltage[msg.sender] = 100;
98:        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
99:    }
```

```diff
diff --git a/src/VoltageManager.sol b/src/VoltageManager.sol
index 61aae93..c857103 100644
--- a/src/VoltageManager.sol
+++ b/src/VoltageManager.sol
@@ -95,7 +95,7 @@ contract VoltageManager {
         require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
         _gameItemsContractInstance.burn(msg.sender, 0, 1);
         ownerVoltage[msg.sender] = 100;
-        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
+        emit VoltageRemaining(msg.sender, 100);
     }

     /// @notice Spends voltage.
```
```
Estimated gas saved: 100 gas units
```



## [G-10] Using calldata instead of memory for read-only arguments in external functions saves gas


### 2 Instances
1. #### Refactor the `addAttributeDivisor` function to use `calldata` in place of `memory` for the `attributeDivisors` parameter.
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L68

Since the `attributeDivisors` array is not modified, we can reduce the gas cost of the `addAttributeDivisor` function by using `calldata` in place of `memory` in the defination of the `attributeDivisors` parameter. In implementing this we would avoid having to copy the array from calldata to memory. The diff below shows how the code should be refactored: 

```solidity
file: src/AiArenaHelper.sol

68:    function addAttributeDivisor(uint8[] memory attributeDivisors) external {    //@audit use calldata in-place memory
69:        require(msg.sender == _ownerAddress);
70:        require(attributeDivisors.length == attributes.length);
71:
72:        uint256 attributesLength = attributes.length;
73:        for (uint8 i = 0; i < attributesLength; i++) {
74:            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
75:        }
76:    }
```

```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..a4ad27d 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -65,7 +65,7 @@ contract AiArenaHelper {

     /// @notice Add attribute divisors for attributes.
     /// @param attributeDivisors An array of attribute divisors.
-    function addAttributeDivisor(uint8[] memory attributeDivisors) external {
+    function addAttributeDivisor(uint8[] calldata attributeDivisors) external {
         require(msg.sender == _ownerAddress);
         require(attributeDivisors.length == attributes.length);
```

2. #### Refactor the `addAttributeProbabilities` function to use `calldata` in place of `memory` for the `probabilities` parameter.
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131

Since the `probabilities` array is not modified, we can reduce the gas cost of the `addAttributeProbabilities` function by using `calldata` in place of `memory` in the defination of the `probabilities` parameter. In implementing this we would avoid having to copy the array from calldata to memory. The diff below shows how the code should be refactored: 

```solidity
file: src/AiArenaHelper.sol

131:    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {    //@audit use calldata in-place memory
132:        require(msg.sender == _ownerAddress);
133:        require(probabilities.length == 6, "Invalid number of attribute arrays");
134:
135:        uint256 attributesLength = attributes.length;
136:        for (uint8 i = 0; i < attributesLength; i++) {
137:            attributeProbabilities[generation][attributes[i]] = probabilities[i];
138:        }
139:    }
```

```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..74c9242 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -128,7 +128,7 @@ contract AiArenaHelper {
      /// @dev Only the owner can call this function.
      /// @param generation The generation number.
      /// @param probabilities An array of attribute probabilities for the generation.
-    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
+    function addAttributeProbabilities(uint256 generation, uint8[][] calldata probabilities) public {
         require(msg.sender == _ownerAddress);
         require(probabilities.length == 6, "Invalid number of attribute arrays");
```



## [G-11]  Non efficient zero initialization
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.
#### Please note this instance was not included in the bots report.

### Proof of concept
Using a simple Remix test

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;


contract InitializeDefaultValue {
    
    uint8[2] public variable = [0,0];

}
```
```
Deployment gas cost: 52959
```


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract NoInitializeDefaultValue {
    
    uint8[2] public variable;

}
```
```
Deployment gas cost: 38891
```


### 1 Instance

1. #### Initalizing variable `generation` to default values is redundant
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L42
```solidity
file: src/FighterFarm.sol

42: uint8[2] public generation = [0, 0];
```

```diff
diff --git a/src/FighterFarm.sol b/src/FighterFarm.sol
index 06ee3e6..64f5c1e 100644
--- a/src/FighterFarm.sol
+++ b/src/FighterFarm.sol
@@ -39,7 +39,7 @@ contract FighterFarm is ERC721, ERC721Enumerable {
     uint256 public rerollCost = 1000 * 10**18;

     /// @notice Stores the current generation for each fighter type.
-    uint8[2] public generation = [0, 0];
+    uint8[2] public generation;

     /// @notice Aggregate number of training sessions recorded.
     uint32 public totalNumTrained;
```
```
Estimated gas saved: 14068 gas units
```


## [G-12] Refactor functions to reduce number of storage reads `SLOAD`
The following functions can be re-written to reduce the number of storage reads in the function.

#### Please note the following instances were not included in the bots reports

### 1 Instances
1. #### Refactor `FighterFarm.incrementGeneration()` to reduce the number of `SLOAD` in the function
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L129-#L134

We can refactor the `incrementGeneration()` function to reduce the number of `SLOAD`. We can do this by creating a new variable and cache the value of `generation[fighterType] + 1` to that variable then assign the variable to the `generation[fighterType]` and also use the variable in the return statement. In implementing this we reduce the `SLOAD` of `generation[fighterType]` from 2 to 1 thereby saving `97` gas units by replacing `SLOAD`(Gwarmacess) `100` gas units with a cheaper stack read `3` gas units. The diff below shows how the code could be re-factored:

```solidity
file: src/FighterFarm.sol

129:    function incrementGeneration(uint8 fighterType) external returns (uint8) {
130:        require(msg.sender == _ownerAddress);
131:        generation[fighterType] += 1;
132:        maxRerollsAllowed[fighterType] += 1;
133:        return generation[fighterType];
134:    }
```

```diff
diff --git a/src/FighterFarm.sol b/src/FighterFarm.sol
index 06ee3e6..535ac3c 100644
--- a/src/FighterFarm.sol
+++ b/src/FighterFarm.sol
@@ -128,9 +128,10 @@ contract FighterFarm is ERC721, ERC721Enumerable {
     /// @return Generation count of the fighter type.
     function incrementGeneration(uint8 fighterType) external returns (uint8) {
         require(msg.sender == _ownerAddress);
-        generation[fighterType] += 1;
+        uint8 fighterTypeGenerationIncremented = generation[fighterType] + 1;
+        generation[fighterType] = fighterTypeGenerationIncremented;
         maxRerollsAllowed[fighterType] += 1;
-        return generation[fighterType];
+        return fighterTypeGenerationIncremented;
     }
```
```
Estimated gas saved: 97 gas units.
```

## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.
