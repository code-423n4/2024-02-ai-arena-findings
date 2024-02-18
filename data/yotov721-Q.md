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

## [L-5] Add arrays length check in `MergingPool::claimRewards`
The function is used to claim multiple rewards and arrays of NFT fighter traits and parameters are passed, such as custom attributes, model type, model URI, but it is not checked that the array sizes are the same and also equal to the unclaimed rewards count. 
### Recommendation: 
Add the following checks
```diff
        uint256 winnersLength;
        uint32 claimIndex = 0;
        uint32 lowerBound = numRoundsClaimed[msg.sender];


+        require(modelURIs.length == modelTypes.length);
+        require(modelTypes.length == customAttributes[2].length);
+        require(customAttributes[2].length == this.getUnclaimedRewards(msg.sender));

        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
                    claimIndex += 1;
                }
            }
        }
```

## [L-6] False return value of ERC20 transfer not handled - multiple occurrences
Would recommend to add an else clause which reverts. This would not be a problem, but in some cases there are state changes done before the transfer which would persist while the ones done in the if clause would not execute and the whole transaction would succeed leaving the protocol in an undesired state. 

For example in: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L283-L289 it would update the user and token staked amounts, but funds would not be transferred and the transaction would succeed, user loosing funds.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L493-L497


## [N-1] Do not pass random bytes in mint function as it can lead to unexpected behavior
The `GameItems::mint` function is used to mint game items. After some checks and operations in calls the default `_mint` function and passes 'random' bytes `_mint(msg.sender, tokenId, quantity, bytes("random"));`
### Recommendation:
Use `_mint(msg.sender, tokenId, quantity, "");` instead

## [N-2] Wrong natspec
In `MergingPool.sol`  The mapping is actually tokenId to fighter points
```
/// @notice Mapping of address to fighter points. 
mapping(uint256 => uint256) public fighterPoints;
```

