# QA Report

## Low Severity Issues

### L-1 - Users redeeming a mint pass can mint Icon fighters with any `iconsType`, including inexisting ones

#### Impact

It is possible to mint Icons via the minting pass with any `iconsType`, including ones that the project was not expecting.

#### Vulnerability Details

`iconsType` is used to determine special traits for the fighter:

```solidity
    if (
        i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
        i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
        i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
    ) {
        finalAttributeProbabilityIndexes[i] = 50;
```

[AiArenaHelper.sol#L100-L105](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L100-L105)

They are not expected to be any number, but within a given range. There is no validation to prevent that as will be proved on the following POC.

#### Proof of Concept

- Add the test to `test/FighterFarm.t.sol`
- Run `forge test --mt "testRedeemMintPassIconMaxValue"`

```solidity
function testRedeemMintPassIconMaxValue() public {
    // Claim mint pass and approve it to _fighterFarmContract
    uint8[2] memory numToMint = [1, 0];
    bytes memory signature = abi.encodePacked(hex"20d5c3e5c6b1457ee95bb5ba0cbf35d70789bad27d94902c67ec738d18f665d84e316edf9b23c154054c7824bba508230449ee98970d7c8b25cc07f3918369481c");
    string[] memory _tokenURIs = new string[](1);
    _tokenURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
    _mintPassContract.claimMintPass(numToMint, signature, _tokenURIs);
    _mintPassContract.approve(address(_fighterFarmContract), 1);

    // Build parameters to pass to redeemMintPass() 
    uint256[] memory _mintpassIdsToBurn = new uint256[](1);
    string[] memory _mintPassDNAs = new string[](1);
    uint8[] memory _fighterTypes = new uint8[](1);
    uint8[] memory _iconsTypes = new uint8[](1);
    string[] memory _neuralNetHashes = new string[](1);
    string[] memory _modelTypes = new string[](1);

    _mintpassIdsToBurn[0] = 1;
    _mintPassDNAs[0] = "dna";
    _fighterTypes[0] = 0;
    _neuralNetHashes[0] = "neuralnethash";
    _modelTypes[0] = "original";
    _iconsTypes[0] = 255;                   // @audit Icon with max value

    // The fighter was successfully minted with _iconsTypes = 255
    _fighterFarmContract.redeemMintPass(
        _mintpassIdsToBurn, _fighterTypes, _iconsTypes, _mintPassDNAs, _neuralNetHashes, _modelTypes
    );

    // Verify that the fighter is the Icon 255
    (,,,,,,, uint8 iconsType,) = _fighterFarmContract.fighters(0);
    assertEq(iconsType, 255);
}
```

#### Recommendation

- Keep a track of the max possible value for Icons (by setting it via an admin function for example)
- Validate that the minted fighter does not exceed that value

### L-2 - Precision error in `curStakeAtRisk`

`curStakeAtRisk` will be 0 when `amountStaked[tokenId] + stakeAtRisk < 1000`:

```solidity
curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
```

[RankedBattle.sol#L439](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L439)

This will make the player lose 0 points when they lose:

```solidity
    bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
    if (success) {
        _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
        amountStaked[tokenId] -= curStakeAtRisk;
```

[RankedBattle.sol#L493-L496](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L493-L496)

But they will still earn points when they win:

```solidity
    points = stakingFactor[tokenId] * eloFactor;
```

[RankedBattle.sol#L445](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L445)

#### Recommendation 

Require that the fighter has some stake at risk:

```diff
    curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
+   require(curStakeAtRisk > 0);
```

### L-3 - Avoid using `_setupRole()` directly

Contracts are using the internal `_setupRole()`, which [is discouraged by OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.5/contracts/access/AccessControl.sol#L195-L202).

Instances: [`addMinter()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L95), [`addStaker()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L103), [`addSpender()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L111).

#### Recommendation

Use `_grantRole()`  instead of `_setupRole()`

### L-4 - Set a range for `getFighterPoints()`

[`getFighterPoints()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L205) only specifies a `maxId`. If the amount of tokens is very large, it may revert with an Out of Gas error

#### Recommendation

Instead of using `maxId`, set an initial id, and an end id, so that it can be queried in a range

### L-5 - Validate `fighterType`

There are instances where `fighterType` can be passed to the `FighterFarm` contract and will fail with an Out of Bounds error when doing `generation[fighterType]` as `generation` [is a two elements length array](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L42), meaning that in those cases `fighterType` can only have values `0`, or `1`.

#### Recommendation

Consider validating the `fighterType` value to return a proper error message. It also may prevent any possible issue if code is updated.

Instances: 

- [`reRoll()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L385)
- [`_createNewFighter()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L512)
- [`createFighterBase()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L470)
- [`incrementGeneration()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L131)

### L-6 - DOS of `updateModel()`

`FighterFarm::updateModel()` [increases the global `totalNumTrained` by one](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L294) each time anyone calls the function.

`totalNumTrained` is [declared as a `uint32`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L45).

A `uint32` has a max value of `4_294_967_295` is a considerable number, but an adversary could spend sufficient resources to call the function repeatedly until the max value is reached (especially considering that it will be deployed on Arbitrum, with lower fees than mainnet).

This will lead to a DOS for all other users, as it is a shared global variable, and it would revert due to an overflow.

#### Recommendation

Declare `totalNumTrained` with a bigger type (like `uint256`). There shouldn't be any drawback in doing so.

### L-7 - Missing check for `iconsTypes.length`

Mainly for consistency, as it [would fail with Out of Bounds](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L258) otherwise.

```solidity
    require(
        mintpassIdsToBurn.length == mintPassDnas.length && 
        mintPassDnas.length == fighterTypes.length && 
        fighterTypes.length == modelHashes.length &&
        modelHashes.length == modelTypes.length
    );
```

[FighterFarm.sol#L243-L248](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L243-L248)

#### Recommendation

```diff
    require(
        mintpassIdsToBurn.length == mintPassDnas.length && 
        mintPassDnas.length == fighterTypes.length && 
        fighterTypes.length == modelHashes.length &&
+       iconsTypes.length == modelHashes.length &&
        modelHashes.length == modelTypes.length
    );
```

### L-8 - `redeemMintPass()` allows to precalculate the dna of fighters to be minted

#### Impact

This can be an unfair advantage for users with minting passes, as they can mint fighters with the best, or most valuable attributes.

Note: Assessed as a Low since it seems to be a business decision, but the impact could be Medium considering the unfair advantage over other players

#### Vulnerability Details

The dna is calculated as a hash of a `mintPassDnas` value that is passed by parameter to the function by the user:

```solidity
uint256(keccak256(abi.encode(mintPassDnas[i]))), 
```

[FighterFarm.sol#L254](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L254)

This means the user can precalculate the outcome and mint the rarest ones.

#### Recommendation

Consider limiting the ability of mint pass holders to mint a fighter with any DNA they want, as it is the same as creating a fighter by passing the expected attributes.

One way to achieve this could be to define the `mintPassDna` on the minting pass contract upon minting of the pass.

### L-9 - Roles can't be revoked

Some roles in `Neuron.sol` can be granted but can't be revoked, as there is not function to do so, there is no assigned admin, nor the `DEFAULT_ADMIN_ROLE` is configured. Instances: [`addMinter()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L95), [`addStaker()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L103), [`addSpender()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L111).

This can be particularly dangerous if some account is compromised, like one with the `MINTER_ROLE`, which will allow minting tokens to any other user.

Other instances are the `GameItems`, which [assigns a not revokable burning address](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L187), and `FighterFarm` which [assigns a staker address (also not revokable)](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L141).

#### Recommendation

- Assign admin roles, the `DEFAULT_ADMIN_ROLE`, or create specific functions to revoke roles for `Neuron.sol`
- Add a boolean to configure `GameItems::setAllowedBurningAddresses()` and `FighterFarm::addStaker()`