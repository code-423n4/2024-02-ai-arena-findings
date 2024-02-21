### Report 1:
#### Missing Comment Description
The comment Description of the redeemMintPass(...) function of the FighterFarm contract shows that all parameters were described but one was missed which is the iconsTypes Array, Ai-Arena should make necessary adjustments to this.
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L236
```solidity
 /// @notice Burns multiple mint passes in exchange for fighter NFTs.
    /// @dev This function requires the length of all input arrays to be equal.
    /// @dev Each input array must correspond to the same index, i.e., the first element in each 
    /// array belongs to the same mint pass, and so on.
    /// @param mintpassIdsToBurn Array of mint pass IDs to be burned for each fighter to be minted.
    /// @param mintPassDnas Array of DNA strings of the mint passes to be minted as fighters.
    /// @param fighterTypes Array of fighter types corresponding to the fighters being minted.
    /// @param modelHashes Array of ML model hashes corresponding to the fighters being minted. 
    /// @param modelTypes Array of ML model types corresponding to the fighters being minted.
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
