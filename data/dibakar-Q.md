## [L-1] `FighterFarm.sol::redeemMintPass` require check doesn't validate the length of `iconsTypes` causes to create a fighter with unbalanced physical attributes.

#### Description
The documentation of `FighterFarm.sol::redeemMintPass` states that this function requires the length of all input arrays to be equal. However, the require check doesn't validate the length of `uint8[] calldata iconsTypes` input array.

[Source of the Issue](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L243)

```solidity
@>	require(
		mintpassIdsToBurn.length == mintPassDnas.length
		&& mintPassDnas.length == fighterTypes.length
		&& fighterTypes.length == modelHashes.length
		&& modelHashes.length == modelTypes.length
	);
```

#### Impact
Bypasses the require statement and proceeds to create a unbalanced Fighter even when the `iconsTypes` input is not valid or the length of the array is not equal with the other input array.

#### Proof of Concept
Add the following test to the `FighterFarm.t.sol` -

```solidity
function test_RedeemMintPassBreakRequire() public {
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
	uint8[] memory _iconsTypes = new uint8[](2);
	string[] memory _neuralNetHashes = new string[](1);
	string[] memory _modelTypes = new string[](1);

	_mintpassIdsToBurn[0] = 1;
	_mintPassDNAs[0] = "dna";
	_fighterTypes[0] = 0;
	_neuralNetHashes[0] = "neuralnethash";
	_modelTypes[0] = "original";
	_iconsTypes[1] = 1;

	// approve the fighterfarm contract to burn the mintpass
	_mintPassContract.approve(address(_fighterFarmContract), 1);

	_fighterFarmContract.redeemMintPass(
		_mintpassIdsToBurn, _fighterTypes, _iconsTypes, _mintPassDNAs, _neuralNetHashes, _modelTypes
	);

	// check balance to see if we successfully redeemed the mintpass for a fighter
	assertEq(_fighterFarmContract.balanceOf(_ownerAddress), 1);
	// check balance to see if the mintpass was burned
	assertEq(_mintPassContract.balanceOf(_ownerAddress), 0);
	// check if the length of all input arrays to be equal
	assertFalse(
		_mintpassIdsToBurn.length == _mintPassDNAs.length && _mintPassDNAs.length == _fighterTypes.length
			&& _fighterTypes.length == _neuralNetHashes.length && _neuralNetHashes.length == _modelTypes.length
			&& _modelTypes.length == _iconsTypes.length
	);
}
```

#### Recommended Mitigation
Consider adding the length of the `iconsTypes` in the require statement for crosschecking with other input arrays length.

```diff
-	require(
-		mintpassIdsToBurn.length == mintPassDnas.length
-		&& mintPassDnas.length == fighterTypes.length
-		&& fighterTypes.length == modelHashes.length
-		&& modelHashes.length == modelTypes.length
-	);
+	require(
+		mintpassIdsToBurn.length == mintPassDnas.length
+		&& mintPassDnas.length == fighterTypes.length
+		&& fighterTypes.length == modelHashes.length
+		&& modelHashes.length == modelTypes.length
+		&& modelTypes.length == iconsTypes.length
+	);
```

## [L-2] Constructors parameter implementation does not match specification.

#### Description
In several contracts in the constructor, the NATSPEC states that the contract deployer should be the `_ownerAddress`. However, in the `constructor` implementaion the `_ownerAddress` is being set from the input paramenters.

Found in -

1. [FightingFarm::constructor](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L104)
2. [GameItems::constructor](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L95C5-L95C16)
3. [MerginPool::constructor](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L71)
4. [Neuron::constructor](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L68)
5. [RankedBattle::constructor](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L146)

#### Recommended Mitigation
Consider setting the `_ownerAddress` directly from the `msg.sender` according to the documentation instead of setting it from a user input to avoid confusion and unwanted accidents.

```diff
-	constructor(address ownerAddress, address rankedBattleAddress, address fighterFarmAddress) {
-        _ownerAddress = ownerAddress;
+	constructor(address rankedBattleAddress, address fighterFarmAddress) {
+        _ownerAddress = msg.sender;
```
