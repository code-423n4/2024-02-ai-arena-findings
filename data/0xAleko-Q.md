1. https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/MergingPool.sol#L206

points array length is 1. If user input maxId more than 1, function will be reverted: Index out of bound.

PoC:
```
function testRevertGetFighterPoints() public {
        // owner mints a fighter by claiming
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
        vm.stopPrank();

        assertEq(_mergingPoolContract.totalPoints(), 300);

        // getFighterPoints for owners fighter
        uint256[] memory fighterPoints = _mergingPoolContract.getFighterPoints(3);
        assertEq(fighterPoints.length >= 1 && fighterPoints[0] == 100, true);
    }
```

Recommendation: 
`-uint256[] memory points = new uint256[](1);`
`+uint256[] memory points = new uint256[](maxId);`