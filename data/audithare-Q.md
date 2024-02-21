## Speculation on future code: Missing checks for ERC20 `transfer()` return value

### Summary

ERC20 `transfer()`'s boolean return value is not properly propagated and checked in two instances in the code:

1. `RankedBattle.setNewRound()` [progresses](https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/RankedBattle.sol#L237-L238) even if the transfer inside [`StakeAtRisk.setNewRound()`](https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/StakeAtRisk.sol#L80-L82) > [`_sweepLostStake()`](https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/StakeAtRisk.sol#L143) fails.
2. `RankedBattle._addResultPoints()` [proceeds](https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/RankedBattle.sol#L461) even if the [transfer inside `StakeAtRisk.reclaimNRN()`](https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/StakeAtRisk.sol#L100-L101) fails.

Currently, this does not manifest in an exploit, as OpenZeppelin's ERC20 implementation [always returns `true`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d/contracts/token/ERC20/ERC20.sol#L116) (unless it reverts).

However, there is no such guarantee in [EIP-20](https://eips.ethereum.org/EIPS/eip-20), nor in the [OpenZeppelin docs](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20#IERC20-transfer-address-uint256-):

> Returns a boolean value indicating whether the operation succeeded.

If `transfer` was to return `false`, this would have the following effects:

1. `setNewRound` will start the new round with the previous rounds' existing stake-at-risk (which may skew stake/point calculation), and eventually sweep the existing stake-at-risk (of the previous and current rounds). 
2. `_addResultPoints` will [keep the stake-at-risk](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/StakeAtRisk.sol#L102-L104), but also [add the same amount to not-at-risk stake](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L462), which can be used to inflate rewards.

### Code locations

* https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/RankedBattle.sol#L237-L238
* https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/RankedBattle.sol#L461
* https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/StakeAtRisk.sol#L80-L82
* https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/StakeAtRisk.sol#L100-L101
* https://github.com/code-423n4/2024-02-ai-arena/blob/68be0262e285c32c126786eeb2915f2207019b15/src/StakeAtRisk.sol#L143

### Recommendation

Propagate `transfer`'s result into the return value, and handle appropriately in the calling code.

## Branches in test code may silently cover failures

### Summary

Some of tests in `FighterFarm.t.sol` contain branches that make parts of the test code conditional.
Future code changes could make these tests silently skip assertions, while the tests still (unexpectedly) succeed.

### Code locations

* https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/test/FighterFarm.t.sol#L294
* https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/test/FighterFarm.t.sol#L308
* https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/test/FighterFarm.t.sol#L327
* https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/test/FighterFarm.t.sol#L345

### Recommendation

Replace `if` branches with assertions:

```diff
diff --git a/test/FighterFarm.t.sol b/test/FighterFarm.t.sol
index afd2f50..6ff90b9 100644
--- a/test/FighterFarm.t.sol
+++ b/test/FighterFarm.t.sol
@@ -291,7 +291,7 @@ contract FighterFarmTest is Test {
     /// @notice Test that the ownerOf tokenId can update the fighter's model.
     function testUpdateModelFromTokenOwner() public {
         _mintFromMergingPool(_ownerAddress);
-        if (_fighterFarmContract.ownerOf(0) == _ownerAddress) {
+        assert(_fighterFarmContract.ownerOf(0) == _ownerAddress);
         uint256 tokenId = 0;
         string memory modelHash = "newModelHash";
         string memory modelType = "newModelType";
@@ -300,12 +300,11 @@ contract FighterFarmTest is Test {
         _fighterFarmContract.updateModel(tokenId, modelHash, modelType);
         assertEq(_fighterFarmContract.numTrained(tokenId), 1);
     }
-    }
 
     /// @notice Test that the non owner of tokenId cannot update the fighter's model.
     function testUpdateModelFromNonTokenOwner() public {
         _mintFromMergingPool(_ownerAddress);
-        if (_fighterFarmContract.ownerOf(0) == _ownerAddress) {
+        assert(_fighterFarmContract.ownerOf(0) == _ownerAddress);
         uint256 tokenId = 0;
         string memory modelHash = "newModelHash";
         string memory modelType = "newModelType";
@@ -316,7 +315,6 @@ contract FighterFarmTest is Test {
         _fighterFarmContract.updateModel(tokenId, modelHash, modelType);
         assertEq(_fighterFarmContract.numTrained(tokenId), 0);
     }
-    }
 
     /// @notice Test paying with neuron to reroll a fighter
     function testReroll() public {
@@ -324,7 +322,7 @@ contract FighterFarmTest is Test {
         // get 4k neuron from treasury
         _fundUserWith4kNeuronByTreasury(_ownerAddress);
         // after successfully minting a fighter, update the model
-        if (_fighterFarmContract.ownerOf(0) == _ownerAddress) {
+        assert(_fighterFarmContract.ownerOf(0) == _ownerAddress);
         uint256 neuronBalanceBeforeReRoll = _neuronContract.balanceOf(_ownerAddress);
         uint8 tokenId = 0;
         uint8 fighterType = 0;
@@ -334,7 +332,6 @@ contract FighterFarmTest is Test {
         assertEq(_fighterFarmContract.numRerolls(0), 1);
         assertEq(neuronBalanceBeforeReRoll > _neuronContract.balanceOf(_ownerAddress), true);
     }
-    }
 
     /// @notice Test that the reroll fails if maxRerolls is exceeded.
     function testRerollRevertWhenLimitExceeded() public {
@@ -342,7 +339,7 @@ contract FighterFarmTest is Test {
         // get 4k neuron from treasury
         _fundUserWith4kNeuronByTreasury(_ownerAddress);
         // after successfully minting a fighter, update the model
-        if (_fighterFarmContract.ownerOf(0) == _ownerAddress) {
+        assert(_fighterFarmContract.ownerOf(0) == _ownerAddress);
         uint8 maxRerolls = _fighterFarmContract.maxRerollsAllowed(0);
         uint8 exceededLimit = maxRerolls + 1;
         uint256 neuronBalanceBeforeReRoll = _neuronContract.balanceOf(_ownerAddress);
@@ -360,7 +357,6 @@ contract FighterFarmTest is Test {
             }
         }
     }
-    }
 
     /// @notice Test that tokenId exists after minting a fighter.
     function testDoesTokenExists() public {
```

## GameItems tests setup is incorrect and incomplete

The [GameItems.t.sol::setUp()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/test/GameItems.t.sol#L19-L31) function is incorrect, in that it sets up the only game item (battery) with incorrect restrictions, in particular with finite supply, and with allowed transferability. Compare:

- [GameItems.t.sol::setUp()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/test/GameItems.t.sol#L30)
  ```_gameItemsContract.createGameItem("Battery", "https://ipfs.io/ipfs/", true, true, 10_000, 1 * 10 ** 18, 10);``` 
- [Deployer.s.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/script/Deployer.s.sol#L56-L64):
  ```gameItems.createGameItem("battery", "https://ipfs.io/ipfs/...", false, false, 0, 10 * 10 ** 18, 5);```

Testing the contracts with the values that are opposite to what is being deployed may and will lead to missed bugs.

Moreover, the testing setup is incomplete, in that it doesn't test for various combinations of possible attributes of game items. E.g. the game items with infinite supply, or restricted transferability, or a small number of items remaining are not tested at all. Without exhaustive testing for possible values of parameters, bugs may again get missed.
