## `RankedBattle.totalBattles` Double Counting
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L345-L348
- The `totalBattles` count inaccurately increments for both initiator and opponent. Recommendation:
```diff
diff --git a/src/RankedBattle.sol b/src/RankedBattle.sol
index 53a89ca..fd2a2bb 100644
--- a/src/RankedBattle.sol
+++ b/src/RankedBattle.sol
@@ -344,8 +344,8 @@ contract RankedBattle {
         }
         if (initiatorBool) {
             _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
+            totalBattles += 1;
         }
-        totalBattles += 1;
     }
```

## Incorrect `globalStakedAmount` Calculation
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L496
- `globalStakedAmount` fails to account for battle risks, leading to inaccurate totals. Recommendation:
```diff
diff --git a/src/RankedBattle.sol b/src/RankedBattle.sol
index 53a89ca..4eb1c8e 100644
--- a/src/RankedBattle.sol
+++ b/src/RankedBattle.sol
@@ -460,6 +460,7 @@ contract RankedBattle {
             if (curStakeAtRisk > 0) {
                 _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
                 amountStaked[tokenId] += curStakeAtRisk;
+                globalStakedAmount -= curStakeAtRisk;
             }
@@ -494,6 +495,7 @@ contract RankedBattle {
                 if (success) {
                     _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                     amountStaked[tokenId] -= curStakeAtRisk;
+                    globalStakedAmount -= curStakeAtRisk;
                 }
             }
```

## `AiArenaHelper.dnaToIndex` Documentation
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L165-L168
- The function description for `dnaToIndex` is misleading. Recommendation:
```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..5034500 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -162,9 +162,10 @@ contract AiArenaHelper {
         return attributeProbabilities[generation][attribute];
     }    
-     /// @dev Convert DNA and rarity rank into an attribute probability index.
-     /// @param attribute The attribute name.
+     /// @dev Extract attribute probability
+     /// @param generation The generation number.
      /// @param rarityRank The rarity rank.
+     /// @param attribute The attribute name.
      /// @return attributeProbabilityIndex attribute probability index.
     function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
         public 
```

## Inflexible `_delegatedAddress` in `FighterFarm`
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L108
- The inability to change `_delegatedAddress` limits administrative actions. Recommendation:
```diff
diff --git a/src/FighterFarm.sol b/src/FighterFarm.sol
index 06ee3e6..20faee0 100644
--- a/src/FighterFarm.sol
+++ b/src/FighterFarm.sol
@@ -173,6 +173,14 @@ contract FighterFarm is ERC721, ERC721Enumerable {
         _mergingPoolAddress = mergingPoolAddress;
     }
+    /// @notice Sets delegated address.
+    /// @dev Only the owner address is authorized to call this function.
+    /// @param delegatedAddress Address of the new delegated party.
+    function setDelegatedAddress(address delegatedAddress) external {
+        require(msg.sender == _ownerAddress);
+        _delegatedAddress = delegatedAddress;
+    }
```

## No Event on `reRoll` URL Clearing in `FighterFarm`
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L389
- Clearing a token URL during `reRoll` without an event disrupts market visibility. Set new URL as soon as possible. Recommendation:
```diff
diff --git a/src/FighterFarm.sol b/src/FighterFarm.sol
index 06ee3e6..33e4743 100644
--- a/src/FighterFarm.sol
+++ b/src/FighterFarm.sol
@@ -387,6 +387,7 @@ contract FighterFarm is ERC721, ERC721Enumerable {
                 fighters[tokenId].dendroidBool
             );
             _tokenURIs[tokenId] = "";
+            emit FighteUrlCleared(id);
         }
     }    
```

## Front-running `RankedBattle.updateBattleRecord()`
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L313-L349
- The potential for front-running updateBattleRecord() could allow users to unfairly increase stakes on winning battles. Recommendation: Consider mechanisms to prevent stake changes during battle processing, such as admin controls to lock staking during critical periods.

## Invalid Data Type in `AiArenaHelper.addAttributeProbabilities`
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131
- `generation` should consistently use `uint8` for compatibility. Recommendation:
```diff
diff --git a/src/AiArenaHelper.sol b/src/AiArenaHelper.sol
index b93dde8..851a3ad 100644
--- a/src/AiArenaHelper.sol
+++ b/src/AiArenaHelper.sol
@@ -128,7 +128,7 @@ contract AiArenaHelper {
      /// @dev Only the owner can call this function.
      /// @param generation The generation number.
      /// @param probabilities An array of attribute probabilities for the generation.
-    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
+    function addAttributeProbabilities(uint8 generation, uint8[][] memory probabilities) public {
         require(msg.sender == _ownerAddress);
         require(probabilities.length == 6, "Invalid number of attribute arrays");
```

## `MAX_SUPPLY` unreachable in `Neuron.mint`
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L156
- Recommendation:
```diff
diff --git a/src/Neuron.sol b/src/Neuron.sol
index d400a49..cb34441 100644
--- a/src/Neuron.sol
+++ b/src/Neuron.sol
@@ -153,7 +153,7 @@ contract Neuron is ERC20, AccessControl {
     /// @param to The address to which the tokens will be minted.
     /// @param amount The amount of tokens to be minted.
     function mint(address to, uint256 amount) public virtual {
-        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
+        require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply");
         require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
         _mint(to, amount);
     }
```