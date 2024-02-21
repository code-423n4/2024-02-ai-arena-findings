## [L-01] Repeatedly assign values to variables

attributeProbabilities arrays assigns values twice in constructor of AiArenaHelper.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L49

```solidity
    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L137

```solidity
    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```

## [L-02] Uncheck arrays length 

there is no check whether the iconsTypes length is equal to the length of other arrays in redeemMintPass of FighterFarm:

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L244-L248


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

