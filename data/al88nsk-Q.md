## L-1: 'updateBattleRecord' from contract 'RankedBattle' can be front-run if the app is deployed on a chain with mempool

#### Description:
The app is going to be deployed to Arbitrum, which do not use mempool and is not vulnerable to front-running. However, the project team is not decided whether the app will be deployed to other chains. In case the app is deployed to a chain with mempool, the function [updateBattleRecord](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L322) can be front-run. A fighter owner can stake additional amount or unstake a fighter depending on the battle result.

## L-2: If NRN transfer fails, the changes made are not reverted

#### Description:
Function [unstakeNRN](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L270) decreases `amountStaked[tokenId]` and `globalStakedAmount`, updates `stakingFactor[tokenId]` and then transfers NRNs. Then it checks the result of transfer, and calls `_fighterFarmInstance.updateFighterStaking(tokenId, false);` only in case when the result is success. However, the other changes are already made and in case the result is not success, the state of the contract becomes inconsistent. The user will not be able to unstake the fighter again, because staked amount was changed.

## L-3: New `roundId` value is not validated

#### Description:
Function [setNewRound](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L78) does not validate if the `roundId_` corresponds to any existing round. If due to some error the `roundId_` value corresponds to an existing round, the values from mappings `totalStakeAtRisk` and `stakeAtRisk` will be reused and the calculations will be incorrect.