## [L-1] `MergingPool::getFighterPoints()` reverts if max id is more than 1

### Description: 
The `getFighterPoints` function returns accumulated points for multiple fighters up to a specific maximum id. To do so it creates an array in memory with size of 1. Note that memory arrays in solidity are not resizable. This will cause the function to revert for any `maxId > 1`. 

### PoC:
Add the following test to `MergingPool.t.sol`
```javascript
    function testGetFighterPointsOutOfBounds() public {
        // owner mints a fighter by claiming
        _mintFromMergingPool(_ownerAddress);
        address owner = _fighterFarmContract.ownerOf(0);
        assertEq(owner, _ownerAddress);

        // rankedeBattle contract adds points
        vm.prank(address(_rankedBattleContract));
        _mergingPoolContract.addPoints(0, 100);
        _mergingPoolContract.addPoints(1, 100);
        _mergingPoolContract.addPoints(2, 100);
        vm.stopPrank();

        // getFighterPoints for multiple fighter
        vm.expectRevert(stdError.indexOOBError);
        _mergingPoolContract.getFighterPoints(3);
    }
```

### Recomendation: 
Set array size to `maxId` instead of 1
```diff
    function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
+        uint256[] memory points = new uint256[](maxId);
-        uint256[] memory points = new uint256[](1);
        for (uint256 i = 0; i < maxId; i++) {
            points[i] = fighterPoints[i];
        }
        return points;
    }
```

## [L-2] Add 0 check in `AiArenaHelper::addAttributeDivisor()`
 Recommended to add 0 checks for `defaultAttributeDivisor[i]` in `AiArenaHelper::addAttributeDivisor` since it can lead to division by 0 error and  block users from creating new fighters or rerolling existing ones. The `attributeToDnaDivisor` is used to calculate the rarity rank by dividing the dna.

## [L-3] PUSH0 opcode not supported on Arbitrum
The contracts use solidity version `>=0.8.0 <0.9.0`, but in version `0.8.20` the PUSH0 opcode is introduced. This opcode is not present on Arbitrum.
It is recommended to set the compiler version to
`0.8.19` or earlier, to prevent any mistakes with the evm version in the future.

## [L-4] Set max Bps loss in `RankedBattle::setBpsLostPerLoss`
The max value of basis points by default is 10_000 and is currently set to 0.1 % = 10 bps. This can be changes in `RankedBattle::setBpsLostPerLoss()` only by the admin. It would be really nice to have a check that the new bsp lost per round are less than 10_000. 
