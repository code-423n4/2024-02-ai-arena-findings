## Issue-1
`RankedBattle.updateBattleRecord()` accounts twice a single battle, once when results are pushed for the initiator, and once for the opponent. This could be problematic if the game server logic depends on an accurate reading of `totalBattles`

**Links to code**
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L348

**Fix**
```diff
@@ -344,8 +344,8 @@ contract RankedBattle {
         }
         if (initiatorBool) {
             _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
+            totalBattles += 1;
         }
-        totalBattles += 1;
     }
```

## Issue-2
There is duplication of constructor logic in `AiArenaHelper.sol`. There is no need to call `addAttributeProbabilities()`, the for loop will set them regardless. Only the length check from `addAttributeProbabilities()` should be extracted in the constructor.

```solidity
constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // Initialize the probabilities for each attribute
    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
        attributeProbabilities[0][attributes[i]] = probabilities[i];
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
} 
```


**Links to code**
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L45


**Fix**
```diff
@@ -42,7 +42,7 @@ contract AiArenaHelper {
         _ownerAddress = msg.sender;
 
         // Initialize the probabilities for each attribute
-        addAttributeProbabilities(0, probabilities);
+        require(probabilities.length == 6, "Invalid number of attribute arrays");
```

## Issue-3
Currently the `rerollCost` in `FighterFarm.sol` is hard-coded. Consider including a setter function that can alter it, depending on the price of $NRN it could become too expensive or cheap to re-roll.

**Links to code**
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L39

**Fix**
```diff
@@ -181,6 +181,10 @@ contract FighterFarm is ERC721, ERC721Enumerable {
         require(msg.sender == _delegatedAddress);
         _tokenURIs[tokenId] = newTokenURI;
     }
+    function setRerollCost(uint256 amount) external {
+        require(msg.sender == _ownerAddress);
+        rerollCost = amount;
+    }
```

## Issue-4
There is logic redundancy in `GameItems.mint()`. We either need `allGameItemAttributes[tokenId].finiteSupply == false` to be `false` or to have items remaining i.e. `quantity <= allGameItemAttributes[tokenId].itemsRemaining` is true.
```solidity
require(
    allGameItemAttributes[tokenId].finiteSupply == false || 
    (   
        // @audit this is redundant redundancy
        allGameItemAttributes[tokenId].finiteSupply == true && 
        quantity <= allGameItemAttributes[tokenId].itemsRemaining
    )
);
```

**Links to code**
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L154

**Fix**
```diff
@@ -151,7 +151,6 @@ contract GameItems is ERC1155 {
         require(
             allGameItemAttributes[tokenId].finiteSupply == false || 
             (
-                allGameItemAttributes[tokenId].finiteSupply == true && 
                 quantity <= allGameItemAttributes[tokenId].itemsRemaining
             )
         );
```

## Issue-5
Redundant allowance check in `Neuron.claim()`. The functions does an allowance check even though OZ ERC20 `transferFrom` does the same check in the internal `_spendAllowance`.
```solidity
function claim(uint256 amount) external {
    require(
        allowance(treasuryAddress, msg.sender) >= amount, 
        "ERC20: claim amount exceeds allowance"
    );
    transferFrom(treasuryAddress, msg.sender, amount);
    emit TokensClaimed(msg.sender, amount);
}
```
**Links to code**
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L140
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d/contracts/token/ERC20/ERC20.sol#L337

**Fix**
```diff
@@ -136,10 +138,6 @@ contract Neuron is ERC20, AccessControl {
     /// @notice Claims the specified amount of tokens from the treasury address to the caller's address.
     /// @param amount The amount to be claimed
     function claim(uint256 amount) external {
-        require(
-            allowance(treasuryAddress, msg.sender) >= amount, 
-            "ERC20: claim amount exceeds allowance"
-        );
         transferFrom(treasuryAddress, msg.sender, amount);
         emit TokensClaimed(msg.sender, amount);
```