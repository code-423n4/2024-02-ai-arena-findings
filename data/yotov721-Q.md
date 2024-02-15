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