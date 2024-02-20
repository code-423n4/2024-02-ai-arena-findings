## Title

Global Staked Amount Not Updated on Reclaim in RankedBattle Contract

## Impact

The failure to update the `globalStakedAmount` variable when stakes are reclaimed introduces inconsistencies in the contract's state. This oversight can lead to inaccurate tracking of the total amount staked within the contract, affecting the integrity of staking logic and potentially the overall game mechanics.

## Proof of Concept

In `RankedBattle.sol:461`, when the `curStakeAtRisk` amount is reclaimed by a user (putting NRN back into their staking pool), the `amountStaked[tokenId]` is correctly updated with `curStakeAtRisk`. However, the contract fails to update the `globalStakedAmount`, which should reflect the total NRN staked across all users. This omission means that actions relying on the `globalStakedAmount` for calculations or validations may operate on stale or incorrect data, leading to potential exploits or unfair advantages.

## Tools Used

Manual Review

## Recommended Mitigation Steps

1. Ensure that `globalStakedAmount` is accurately updated whenever stakes are reclaimed or altered. This involves adding a line to increment or decrement `globalStakedAmount` accordingly within the same conditional block where `amountStaked[tokenId]` is updated.

## Issue Type

Other

--------------------------

## Title

Reroll Function Vulnerability in FighterFarm Contract Due to Missing FighterType Check

## Impact

This vulnerability could allow users to bypass reroll limits for different fighter types by not enforcing a check on the `fighterType` parameter. It potentially enables exploiting the reroll mechanism, leading to imbalance in gameplay and unfair advantages. Additionally, passing an invalid `fighterType` greater than allowed values can cause the function to revert, affecting the usability and stability of the contract.

## Proof of Concept

In `FighterFarm.sol:370`, the `reRoll` function lacks a validation check for the `fighterType` parameter. This omission means users could pass a `fighterType` that does not match the type of the actual fighter represented by the tokenId, allowing them to either bypass reroll limits (if `maxRerollsAllowed` differs across types) or trigger a function revert by passing an unsupported `fighterType`.

## Tools Used

Manual Review

## Recommended Mitigation Steps

1. Implement a check within the `reRoll` function to validate the `fighterType` against the allowed range or set of values, ensuring it corresponds to a legitimate category of fighters.
2. Consider setting a global limit for rerolls that applies across all fighter types to simplify validation logic and prevent bypassing through type manipulation.
3. Implement error handling for invalid `fighterType` inputs to ensure the contract fails gracefully without reverting transactions unnecessarily.

## Issue Type

Invalid Validation

