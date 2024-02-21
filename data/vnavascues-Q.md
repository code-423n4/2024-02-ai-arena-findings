# QA Report

## Low Risk Issues

### L-01 Ownership is transferred in a single step

**Instances**

- [AiArenaHelper.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L61)
- [FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L120)
- [GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L108)
- [MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L89)
- [Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85)
- [RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L167)
- [VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L64)

**Description**

Function `transferOwnership` transfers the ownership of the smart contract to a new address in a single transaction. This is risky as errors in specifying the new address can result in unintended consequences.

**Recommended Mitigation Steps**

Use two-step transactions to set the new owner address, e.g. by integrating OpenZeppelin [Ownable2Step](https://docs.openzeppelin.com/contracts/5.x/api/access#Ownable2Step).

### L-02 Admin is not adjusted when transferring ownership

**Instances**

- [GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L108)
- [MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L89)
- [Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85)
- [RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L167)
- [VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L64)

**Description**

Contracts that manage administrator permissions always set the `_ownerAddress` as administrator in the `constructor`. However, `transferOwnership` neither revokes the permission from the previous owner, nor sets the new owner as administrator.

All these contracts include a function `adjustAdminAccess` for managing administrator permissions, but decoupling this from when ownership is transferred may lead to errors.

**Recommended Mitigation Steps**

Update `isAdmin` in `transferOwnership`, e.g. by setting the current (soon to be previous) owner address to `false` and the new owner to `true`.

### L-03 GameItems `allowedBurningAddresses` cannot be unset

**Instances**

- [GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L185)

**Description**

Function `setAllowedBurningAddresses` can only grant and never revoke this permission to given addresses since there is a hardcoded `true`.

**Recommended Mitigation Steps**

Remove the hardcoded boolean and, similarly to `adjustAllowedVoltageSpenders`, add a second parameter `bool allowed` to `setAllowedBurningAddresses`, to indicate whether the given address should or should not be allowed to burn game items.

### L-04 FighterFarm `hasStakerRole` cannot be unset

**Instances**

- [FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L139)

**Description**

Function `addStaker` can only grant and never revoke this permission to given addresses since there is a hardcoded `true`.

**Recommended Mitigation Steps**

Remove the hardcoded boolean and, similarly to `adjustAllowedVoltageSpenders`, add a second parameter `bool allowed` to `addStaker`, to indicate whether the given address should or should not be allowed to stake.

### L-05 FighterFarm `redeemMintPass` can revert due to `iconsTypes` access out-of-bounds

**Instances**

- [FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L258)

**Description**

Function `redeemMintPass` requires that all input parameters of type array have the same length except for `iconsTypes`. After that, the function traverses all the arrays in a single `for` loop.

Since the function missed validating the length of `iconsTypes`, it could revert unexpectedly when attempting to access an index out-of-bounds.

**Recommended Mitigation Steps**

Update the `require` [L243](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L243) so that it also checks `iconsTypes.length`.

### L-06 FighterFarm `updateModel` always increments stats, even when receiving same model data

**Instances**

- [FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L283)

**Description**

Function `updateModel` always increments `numTrained[tokenId]` and `totalNumTrained` (+1). It skips validating whether `modelHash` and `modelType` params are different from data currently in storage. This could be used by players to their advantage.

**Recommended Mitigation Steps**

Validate `modelHash` and `modelType`: compare given params against storage and only increment `numTrained[tokenId]` and `totalNumTrained` if there are differences.

### L-07 VoltageManager `spendVoltage` not protected against underflow

**Instances**

- [VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L110)

**Description**

Function `spendVoltage` skips validating whether `ownerVoltage[spender]` is greater or equal than `voltageSpent`. If `voltageSpent` is greater, the subtraction operation [L110](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L110) will cause a revert due to an underflow error.

**Recommended Mitigation Steps**

Validate that `ownerVoltage[spender]` is greater or equal than `voltageSpent` before performing the subtraction.

### L-08 Neuron `setupAirdrop` can interfere with existing approvals

**Instances**

- [Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L127)

**Description**

The Neuron contract might already have certain approvals set for specific addresses. Initiating an airdrop directly from the Neuron contract could potentially override or interfere with these existing approvals, leading to unexpected behavior.

**Recommended Mitigation Steps**

Move the airdrop logic onto a separate contract.

### L-09 Neuron NRN `MAX_SUPPLY` can never be minted

**Instances**

- [Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L156)

**Description**

Function `mint` includes a condition requiring `totalSupply() + amount` to be lesser than `MAX_SUPPLY`, which prevents from minting all the NRN supply.

**Recommended Mitigation Steps**

Use `<=` operator instead of `<` so that all NRN supply can be minted.

### L-10 RankedBattle `unstakeNRN` performs state changes for `amount` `0`

**Instances**

- [RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L270)

**Description**

Function `unstakeNRN` skips validating whether the `amount` of NRN tokens to unstake is not `0`. However, still performs a number of state updates, setting e.g.:

- `stakingFactor[tokenId]`
- `_calculatedStakingFactor[tokenId][roundId] = true`
- `hasUnstaked[tokenId][roundId] = true`

May also emit events, like `Unstaked`.

**Recommended Mitigation Steps**

Validate that the `amount` of NRN tokens to unstake is not `0`.

### L-11 RankedBattle functions not protected against division by zero

**Instances**

- [RankedBattle.sol claimNRN](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L303)
- [RankedBattle.sol getUnclaimedNRN](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L394)

**Description**

Both functions can revert. This is because `totalAccumulatedPoints` for the given round id can be `0`, resulting in a division by zero scenario.

**Recommended Mitigation Steps**

Validate that `totalAccumulatedPoints[roundId]` is not `0` before performing the division.

### L-12 RankedBattle `updateBattleRecord` performs state changes for any `battleResult`

**Instances**

- [RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L322)

**Description**

According to `_updateRecord`, `battleResult` can only adopt 3 values: 0, 1, 2.

Function `updateBattleRecord` skips validating `battleResult`, which results in a number of state updates (points, voltage and `totalBattles`) that may not be correct.

**Recommended Mitigation Steps**

Validate that `battleResult` is within range.

## Non-Critical Issues

### NC-01 AiArenaHelper `attributes` are different from docs

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L17

**Description**

Contract `attributes` are "head", "eyes", "mouth", "body", "hands", "feet".

Documentation attributes ([see The Skin](https://docs.aiarena.io/gaming-competition/nft-makeup#9d1d4411f7814ef0945800431c83dd05)) are "hair", "eyes", "mouth", "body", "hands", "feet".

Notice HEAD vs HAIR.

**Recommended Mitigation Steps**

Review and update fighter attributes accordingly.

### NC-02 AiArenaHelper `constructor` could call `addAttributeDivisor`

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L50

**Description (and Mitigation)**

Contract `constructor` could call `addAttributeDivisor` `external` function on the same contract, instead of updating `attributeToDnaDivisor` mapping. In which case, the function should be made `public`.

### NC-03 FighterFarm calls to `_createNewFighter` and `_createFighterBase` provide weak sources of randomness for `dna` param

**Instances**

- [FighterFarm.sol claimFighters](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L214)
- [redeemMintPass](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L254)
- [mintFromMergingPool](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L324)
- [reRoll](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L379)

**Description**

Fighters `dna` is supposed to be random and unique.

To generate fighters `dna` functions above use Solidity `keccak256` to calculate a hash based on different parameters. The issue is that these parameters are all deterministic and predictable, and as a result `dna` lacks the inherent unpredictability associated with true randomness.

**Recommended Mitigation Steps**

Improve `dna` generation and use a better source of randomness, e.g. [ChainLink VRF](https://docs.chain.link/vrf).

### NC-04 FighterFarm `updateModel` can update model data anytime

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L283

**Description**

Function `updateModel` can be called to update the fighters `modelHash` and `modelType` anytime. This could be used by players to their advantage.

**Recommended Mitigation Steps**

Change `updateModel` so that `modelHash` and `modelType` can only be updated under certain conditions, e.g. after a timelock, after a number of wins, etc.

### NC-05 GameItems `bool` in `struct` `GameItemAttributes` could be `isTransferable`

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L38

**Description (and Mitigation)**

Use `isTransferable`, instead of `transferable`.

### NC-06 GameItems `_mint` hardcoded data

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L173

**Description (and Mitigation)**

Review hardcoded string `random` in `_mint` call.

### NC-07 Functions `instantiate...Contract` have a misleading name

**Instances**

- [FighterFarm.sol instantiateAIArenaHelperContract](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L147)
- [FighterFarm.sol instantiateMintpassContract](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L155)
- [FighterFarm.sol instantiateNeuronContract](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L163)
- [GameItems.sol instantiateNeuronContract](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L139)
- [RankedBattle.sol instantiateNeuronContract](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L201)
- [RankedBattle.sol instantiateMergingPoolContract](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L209)

**Description (and Mitigation)**

Functions above do not create a new contract/instance, they only create a reference. Consider updating function names to e.g. `setNeuronContract`.

### NC-08 RankedBattle `setStakeAtRiskAddress` performs redundant step

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L192

**Description (and Mitigation)**

It is redundant to store a reference to the StakeAtRisk Contract (L195) and the address (L194). Consider removing the address.

### NC-09 StakeAtRisk not providing consistent feedback when reverting

**Instances**

- [StakeAtRisk.sol setNewRound](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L79)
- [reclaimNRN](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L94)
- [updateAtRiskRecords](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L122)

**Description (and Mitigation)**

Functions above validate `msg.sender == _rankedBattleAddress`, but provide mixed feedback. Consider reverting with the same message when checking for a particular condition.

### NC-10 VoltageManager `useVoltageBattery` uses hardcoded `tokenId`

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L93

**Description**

Function `useVoltageBattery` uses a hardcoded `0` when referring to `battery` `tokenId`:

```
require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
_gameItemsContractInstance.burn(msg.sender, 0, 1);
```

This will work for as long as the deployment script keeps creating `battery` as the first game item.

**Recommended Mitigation Steps**

Update the implementation so that it does not rely on hardcoded information.