## the `MergingPools::getFighterPoints` will revert when input is > 0

## Vulnerability details

The function attempts to fetch the points of fighters 0-maxId, but can actually only fetch the points of fighter id 0, because the length of the points array is set to 0. So if we input 1 it will revert with array out of bonds error

## Impact

The impact is not severe as this function is a view function that doesn't impact any logic in the contracts. The function will not display the different fighter points as expected

## POC

https://github.com/code-423n4/2024-02-ai-arena/blob/5b2ab9f9fadd0b91268ff6f22b4ae0fd5b79ec09/src/MergingPool.sol#L207-L211

```solidity
    function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
 @>       uint256[] memory points = new uint256[](1);
        for (uint256 i = 0; i < maxId; i++) {
 @>         points[i] = fighterPoints[i];
        }
```

To prove this I slightly modified the `testGetFighterPoints()` in MegingPool.t.sol.

```
    function testGetFighterPoints() public {
        // owner mints a fighter by claiming
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_ownerAddress);
        address owner = _fighterFarmContract.ownerOf(0);
        assertEq(owner, _ownerAddress);

        // rankedeBattle contract adds points
        vm.startPrank(address(_rankedBattleContract));
        _mergingPoolContract.addPoints(0, 100);
        _mergingPoolContract.addPoints(1, 100);
        _mergingPoolContract.addPoints(2, 100);
        _mergingPoolContract.addPoints(3, 100);
        _mergingPoolContract.addPoints(4, 100);
        _mergingPoolContract.addPoints(5, 100);
        _mergingPoolContract.addPoints(6, 100);
        _mergingPoolContract.addPoints(7, 100);
        assertEq(_mergingPoolContract.totalPoints(), 100);
        // getFighterPoints for owners fighter
        uint256[] memory fighterPoints = _mergingPoolContract.getFighterPoints(8);
        // assertEq(fighterPoints.length >= 1 && fighterPoints[0] == 100, true);
        // getFighterPoints for non existent fighter
        // vm.expectRevert(stdError.indexOOBError);
        // _mergingPoolContract.getFighterPoints(1000);
    }
```

This test will revert with out of bounds array error.

## Mitigation route

In order to mitigate this issue I would modify the design of that function in such way:

```
    function getFighterPoints(uint256[] ids) public view returns(uint256[] memory) {
        uint256[] memory points = new uint256[](Ids.length);
        for (uint256 i = 0; i < ids.length; i++) {
            points[i] = fighterPoints[ids[i]];
        }
        return points;
    }

```  
This slight modification allows msg.sender to fetch the points of the fighters they specified in the array.