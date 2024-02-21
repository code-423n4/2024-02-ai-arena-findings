## [L-1] Unchecked `iconsTypes` Array in `FighterFarm.sol::redeemMintPass` Parameter

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L233C4-L262C6

The `@dev` Natspec comment in the function specifies that all input arrays must be of equal length and correspond to the same index. However, the `iconsTypes` array is not being checked, which allows users to input arbitrary values and lengths that do not correspond with the rest of the arrays.

Add the following test to `FighterFarm.t.sol`:

```solidity
function testRedeemMintPassWithUnmatchedIconTypesLength() public {
        uint8[2] memory numToMint = [1, 0];
        bytes memory signature = abi.encodePacked(
            hex"20d5c3e5c6b1457ee95bb5ba0cbf35d70789bad27d94902c67ec738d18f665d84e316edf9b23c154054c7824bba508230449ee98970d7c8b25cc07f3918369481c"
        );
        string[] memory _tokenURIs = new string[](1);
        _tokenURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";

        // first i have to mint an nft from the mintpass contract
        assertEq(_mintPassContract.mintingPaused(), false);
        _mintPassContract.claimMintPass(numToMint, signature, _tokenURIs);
        assertEq(_mintPassContract.balanceOf(_ownerAddress), 1);
        assertEq(_mintPassContract.ownerOf(1), _ownerAddress);

        // once owning one i can then redeem it for a fighter
        uint256[] memory _mintpassIdsToBurn = new uint256[](1);
        string[] memory _mintPassDNAs = new string[](1);
        uint8[] memory _fighterTypes = new uint8[](1);
        string[] memory _neuralNetHashes = new string[](1);
        string[] memory _modelTypes = new string[](1);

        _mintpassIdsToBurn[0] = 1;
        _mintPassDNAs[0] = "dna";
        _fighterTypes[0] = 0;
        _neuralNetHashes[0] = "neuralnethash";
        _modelTypes[0] = "original";

        // adding unmatched icon types length
        uint8[] memory _iconsTypes = new uint8[](3);
        _iconsTypes[0] = 1;
        _iconsTypes[1] = 2;
        _iconsTypes[2] = 3;

        // approve the fighterfarm contract to burn the mintpass
        _mintPassContract.approve(address(_fighterFarmContract), 1);

        _fighterFarmContract.redeemMintPass(
            _mintpassIdsToBurn, _fighterTypes, _iconsTypes, _mintPassDNAs, _neuralNetHashes, _modelTypes
        );

        // check balance to see if we successfully redeemed the mintpass for a fighter
        assertEq(_fighterFarmContract.balanceOf(_ownerAddress), 1);
        // check balance to see if the mintpass was burned
        assertEq(_mintPassContract.balanceOf(_ownerAddress), 0);
    }
```
This test will still pass because there's no check for the length of iconsTypes.

## [L-2] Incorrect Comparison Operator in `Neuron.sol::mint` Prevents `totalSupply` from Reaching `MAX_SUPPLY`

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L155C1-L159C6

The comparison operator used in the `mint` function is less than (`<`), which prevents `totalSupply` from ever reaching `MAX_SUPPLY`.

```diff
    function mint(address to, uint256 amount) public virtual {
-        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply"); 
+        require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply"); 
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```

## [L-3] Lack of Input Validation for `modelHash` and `modelType` in `FighterFarm::updateModel`

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L283C1-L295C6

The `modelHash` and `modelType` parameters in `FighterFarm::updateModel` are user inputs that lack validation checks. This allows users to input incorrect values, which could lead to the game being unable to read the correct `modelHash` and `modelType`, and consequently, incorrect data being stored on-chain.

```solidity
function updateModel(
        uint256 tokenId, 
        string calldata modelHash,
        string calldata modelType
    ) 
        external
    {
        require(msg.sender == ownerOf(tokenId));
        fighters[tokenId].modelHash = modelHash; //! user can input any arbitrary value
        fighters[tokenId].modelType = modelType; //! user can input any arbitrary value
        numTrained[tokenId] += 1;
        totalNumTrained += 1;
    }
```

*Recommended Mitigation* : 
Consider adding validation checks for `modelHash` and `modelType` to ensure they meet certain criteria before they are assigned to `fighters[tokenId].modelHash` and `fighters[tokenId].modelType` respectively. This could be a predefined list of acceptable values or a pattern that the inputs must match.

## [I-1] Incorrect Natspec Comment for `MergingPool::fighterPoints`

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L50C1-L51C54

```solidity
    /// @notice Mapping of address to fighter points.
    mapping(uint256 => uint256) public fighterPoints;
```

The Natspec comment incorrectly states that the mapping is from an `address` to fighter points. However, the mapping is actually from a `uint256` to another `uint256`.

 