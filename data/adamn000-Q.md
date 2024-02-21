# Missing check in `FighterFarm::redeemMintPass`

There is information that all input arrays should be the same length, but `iconsTypes` input is not checked in require statement.

```javascript
    function redeemMintPass(
        uint256[] calldata mintpassIdsToBurn,
        uint8[] calldata fighterTypes,
        uint8[] calldata iconsTypes,
        string[] calldata mintPassDnas,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) external {
        // q iconsTypes not included in the require statement
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && mintPassDnas.length == fighterTypes.length
                && fighterTypes.length == modelHashes.length && modelHashes.length == modelTypes.length
        );
.
.
.
```