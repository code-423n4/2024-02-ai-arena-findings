## L-01 Return array in `MergingPool::getFighterPoints` prevents the function from working as intended.

### Description

In `MergingPool::getFighterPoints`, The return array is only of size 1, though the comments state the function should return points for all fighters up to maxId:

```solidity

/// @notice Retrieves the points for multiple fighters up to the specified maximum token ID.
/// @param maxId The maximum token ID up to which the points will be retrieved.
/// @return An array of points corresponding to the fighters' token IDs.

```

### Impact

Since the size of the `points` return array is hardcoded to 1, every query to this view function with maxId > 1 will revert with out-of-bounds error.

Replace `testGetFighterPoints` with the following version:

```solidity
function testGetFighterPoints() public {
        // owner mints a fighter by claiming
        _mintFromMergingPool(_ownerAddress);
        address owner = _fighterFarmContract.ownerOf(0);
        assertEq(owner, _ownerAddress);

        // mint a second one
        _mintFromMergingPool(_ownerAddress);
        address owner2 = _fighterFarmContract.ownerOf(1);
        assertEq(owner2, _ownerAddress);

        // rankedeBattle contract adds points
        vm.prank(address(_rankedBattleContract));
        _mergingPoolContract.addPoints(0, 100);
        assertEq(_mergingPoolContract.totalPoints(), 100);

        // add points for second nft
        vm.prank(address(_rankedBattleContract));
        _mergingPoolContract.addPoints(1, 100);
        assertEq(_mergingPoolContract.totalPoints(), 200);

        // getFighterPoints for owners fighter
        uint256[] memory fighterPoints = _mergingPoolContract.getFighterPoints(1);
        assertEq(fighterPoints.length >= 1 && fighterPoints[0] == 100, true);
        
        // getFighterPoints for owners two fighters -> should revert because of bug
        vm.expectRevert(stdError.indexOOBError);
        uint256[] memory fighterPoints2 = _mergingPoolContract.getFighterPoints(2);

        // getFighterPoints for non existent fighter
        vm.expectRevert(stdError.indexOOBError);
        _mergingPoolContract.getFighterPoints(1000);
    }
```

### Recommended Mitigation Steps

Remove the hardcoded value for size of `points` return array.

## L-02 Calculation for weight in `FighterFarm::_createFighterBase` might be incorrect.

### Description

In `FighterFarm::_createFighterBase`, a new set of weight, element and dna is created. According to the protocol, weight should lie in the range 65-95 with jumps of 10 between the 3 weight classes (Striker, Scraper, Slugger), such that the first class is 65-74, second one 75-84, and third one should be 85-94. But the code defines up to 95 as can be seen here:

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L471

```uint256 weight = dna % 31 + 65;```

### Impact

During the creation of a new fighter, the value of weight could be calculated to be 95 which might not be what the protocol intended to.

### Recommended Mitigation Steps

If the intended behavior is as described in the description part of this report, then change the 65 in the equation to 64. Otherwise, it’s the protocol’s decision how to handle this.