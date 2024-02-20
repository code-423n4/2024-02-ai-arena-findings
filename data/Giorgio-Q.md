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


## Deterministic minting allows users to revert the transaction if the NFT attributes don't match their expectations

## Vulnerability details

Users can mint an NFT in several ways. Every mint will have physical attribute that will have different rarity, weight and element. The issue here is that if a user isn't pleased with any of these attributes the transaction can be reverted, and he can mint another time, and repeat this process until satisfied.

## Impact

The randomness is bypassed, user can mint desired object, decreasing their value.

## POC

Since the minting process happens in one transaction only, the whole process can be effectively reverted. The abuser can wait for other people to mint a few NFTs, and this will be enough to change the hash that will impact the minting.

```
            _createNewFighter(
                msg.sender, 
     @>         uint256(keccak256(abi.encode(msg.sender, fighters.length))),
                modelHashes[i], 
                modelTypes[i],
                i < numToMint[0] ? 0 : 1, 
                0,
                [uint256(100), uint256(100)]
```


```
        _createNewFighter(
            to, 
   @>       uint256(keccak256(abi.encode(msg.sender, fighters.length))), 
            modelHash, 
            modelType,
            0, 
            0, 
            customAttributes 
        );
```

Here is an actual implementation that an abuser would use:

```solidity 
 //FighterFarm.t.sol

    function testClaimFightersSelectWhatIwant() public {
        uint8[2] memory numToMint = [1, 0];
        bytes memory claimSignature = abi.encodePacked(
            hex"407c44926b6805cf9755a88022102a9cb21cde80a210bc3ad1db2880f6ea16fa4e1363e7817d5d87e4e64ba29d59aedfb64524620e2180f41ff82ca9edf942d01c"
        );
        string[] memory claimModelHashes = new string[](1);
        claimModelHashes[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";

        string[] memory claimModelTypes = new string[](1);
        claimModelTypes[0] = "original";

        // Right sender of signature should be able to claim fighter

        claimPackage(numToMint, claimSignature, claimModelHashes, claimModelTypes);
    }
    function claimPackage(uint8[2] memory numToMint, bytes memory claimSignature, string[] memory claimModelHashes, string[] memory claimModelTypes) public {
        _fighterFarmContract.claimFighters(numToMint, claimSignature, claimModelHashes, claimModelTypes);
        ( , ,FighterOps.FighterPhysicalAttributes memory physicalAttributes , , , , , ,) = _fighterFarmContract.fighters(0);
        uint256 head = physicalAttributes.head;
        require(head == 1, "head doesn't comply");
    }
```

## Mitigation

This issue is not easy to mitigate. I would consider going through a two step minting process, such as mint dna, have a delay, and then mint nft attribute. Although this transaction will still be revertable, it makes it harder to predict the outcomes, but it is still very possible.
Another more expensive alternative would be to use something like chainlink VRF.