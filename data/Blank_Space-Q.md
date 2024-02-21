#[L-01] Missing length check for `iconsTypes`

The `redeemMintPass` function does not include the `iconsTypes` array in its sanity length checks. This could cause an unintended revert. Consider adding this check as per the diff:

```diff
require(
            mintpassIdsToBurn.length == mintPassDnas.length && mintPassDnas.length == fighterTypes.length
                && fighterTypes.length == modelHashes.length && modelHashes.length == modelTypes.length
++ && iconsTypes.length == modelTypes.length
        );
```
Location: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L243-L248

#[L-02] The `claimRewards` function does not validate the lengths of the parameter arrays

There could be unintended reverts in `claimRewards` due to the fact that the function relies on the input array lengths to be the same (as the index positions are compared), but does not validate this fact. Consider adding a sanity check that compares the array lengths.

```diff
++ require(modelURIs.length == modelTypes.length, "array length mismatch");
```
Location: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139-L167

#[L-03] Players can update the `fighterType` due to lack of validation
A player can change the fighterType by calling `reRoll`. The `reRoll` function takes two parameters namely `tokenId` and `fighterType`, but the function doesn’t compare what the existing fighterType was. Which means that a player could `reRoll` their fighter type if they choose to do so. It will also skew the physicalAttributes of the fighter as the function will call `createPhysicalAttributes` with a false dendroidBool. Consider retrieving the fighterType from the fighters object in stead of passing it as a parameter. 

Location: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L370-L391

#[L-04] The `incrementGeneration` function increases the `maxRerollsAllowed` per fighter type

The `incrementGeneration` function increments the generation per fighter type, but it also increases the `maxRerollsAllowed` for every generation increase, thus the new generation fighters will get an added re-roll every time `incrementGeneration` is called. Also consider that any existing fighter will gain a re-roll with each generation increment with reference to this line in the `reRoll` function: `require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);` as re-rolls are checked on a global level. Consider adding a function that is callable by the owner to specifically increment the re-rolls:

```solidity
   	/// @notice Increases the maxRerolls for all fighters
    /// @dev Only the owner address is authorized to call this function.
    /// @param fighterType Type of fighter either 0 or 1 (champion or dendroid).
    function incrementMaxRerolls(uint8 fighterType) external {
        require(msg.sender == _ownerAddress);
        maxRerollsAllowed[fighterType] += 1;
    }
```
Location: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L129-L134

#[L-05] Admin role changes should be Two step

The owner is extremely powerful in this protocol, but the `transferOwnership` function in various contracts listed below implements a change of ownership with no validation which could be dangerous. Consider implementing a two step “push” and “pull” admin transfer process such as the OpenZeppelin Ownable2Step.sol library,https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol

Locations:

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L61-L64

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L120-L123

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L108-L110

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L89-L92

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85-L88

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L167-L170

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L64-L67

#[I-01] Missing natspec for `icontypes` in`redeemMintPass`

The `redeemMintPass` function is missing natspec for `iconsTypes` . Please consider adding the natspec for the parameter to improve the completeness of the documentation. 

Location: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L233

#[I-02] The following functions do not emit events for storage updates:

#### AiArenaHelper.sol

- `transferOwnership`
- `addAttributeDivisor`
- `addAttributeProbabilities`
- `deleteAttributeProbabilities`

#### FighterFarm.sol

- `transferOwnership`
- `addStaker`
- `instantiateAIArenaHelperContract`
- `instantiateMintpassContract`
- `instantiateNeuronContract`
- `setMergingPoolAddress`
- `setTokenURI`
- `claimFighters`
- `redeemMintPass`
- `updateModel`
- `mintFromMergingPool`
- `reRoll`

#### GameItems.sol

- `transferOwnership`
- `adjustAdminAccess`
- `instantiateNeuronContract`
- `setAllowedBurningAddresses`
- `setTokenURI`

#### MergingPool.sol

- `transferOwnership`
- `adjustAdminAccess`
- `updateWinnersPerPeriod`
- `pickWinner`

#### Neuron.sol

- `transferOwnership`
- `adjustAdminAccess`

#### RankedBattle.sol

- `transferOwnership`
- `adjustAdminAccess`
- `setGameServerAddress`
- `setStakeAtRiskAddress`
- `instantiateNeuronContract`
- `instantiateMergingPoolContract`
- `setRankedNrnDistribution`
- `setBpsLostPerLoss`
- `setNewRound`
- `updateBattleRecord`

#### StakeAtRisk.sol

- `setNewRound`

#### VoltageManager.sol

- `transferOwnership`
- `adjustAdminAccess`
- `adjustAllowedVoltageSpenders`