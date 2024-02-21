## [L-01] function `MergingPool.pickWinner()` has no event
It is recommended to emit events on important functions

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L118-L132

```solidity
    function pickWinner(uint256[] calldata winners) external {
        require(isAdmin[msg.sender]);
        require(winners.length == winnersPerPeriod, "Incorrect number of winners");
        require(!isSelectionComplete[roundId], "Winners are already selected");
        uint256 winnersLength = winners.length;
        address[] memory currentWinnerAddresses = new address[](winnersLength);
        for (uint256 i = 0; i < winnersLength; i++) {
            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
            totalPoints -= fighterPoints[winners[i]];
            fighterPoints[winners[i]] = 0;
        }
        winnerAddresses[roundId] = currentWinnerAddresses;
        isSelectionComplete[roundId] = true;
        roundId += 1;
    }
```

## [L-02] function `FighterFarm.redeemMintPass()` doesn't check the length of the `iconsTypes`.

All inputs must have the same length, and there is a check but the `iconTypes` is missing in the check.

```solidity
    function redeemMintPass(
        uint256[] calldata mintpassIdsToBurn,
        uint8[] calldata fighterTypes,
        uint8[] calldata iconsTypes,
        string[] calldata mintPassDnas,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
            modelHashes.length == modelTypes.length
        );
        for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
            _mintpassInstance.burn(mintpassIdsToBurn[i]);
            _createNewFighter(
                msg.sender, 
                uint256(keccak256(abi.encode(mintPassDnas[i]))), 
                modelHashes[i], 
                modelTypes[i],
                fighterTypes[i],
                iconsTypes[i],
                [uint256(100), uint256(100)]
            );
        }
    }
```