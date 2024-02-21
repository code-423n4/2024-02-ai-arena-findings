# Analysis Report by 0xDetermination

## Game Balance
One of the main concerns I have is that users with zero or tiny stake can initiate battles with users with a huge stake, and cause the loss of a large amount of points without risking a comparable amount of points. This could potentially create issues where users try to grief other players or try to lower the total points accumulated in order to claim a larger portion of the NRN reward pool.

One other concern with the game system is that it might be vulnerable to 'point farming' via a low-elo NFT retraining to a very strong model, and then staking a large amount and winning the majority of the matches until the elo increases to the appropriate level, at which point the stake can be withdrawn and the points kept to earn a large portion of the NRN prize pool.

## Good Points
The sqrt calculation for user `stakingFactor`s is a very nice mechanism to provide diminishing returns for whales and create a more level playing field for the average user.

Although I found many H/M issues in the contract, the majority of them are easily fixed. I don't see many intractable systemic issues or architecture problems in the codebase, which is very good.

## Systemic Issues, Areas of Concern
The one area of concern I have regarding systemic/architecture issues is the synchronization and proper handling of 'stakes at risk' between `RankedBattle.sol` and `StakeAtRisk.sol`. Fighter NFTs should not be transferred with an existing stake, but they can be transferred with existing `stakeAtRisk`; furthermore, when fighters win with an existing `stakeAtRisk` they should have some stake restored. Although the vulnerability mitigation here seems simple at first glance- simply lock NFT transfer when restoring stake from the `stakeAtRisk`- I suspect that this integration is prone to having difficult to spot edge cases.

## Centralization Risks
The centralization risk that can be reduced is the mutability of contract addresses and access control roles.

For example, `GameItems.instantiateNeuronContract()` can change the NRN contract address in `GameItems` at-will, which is a problem if the owner address becomes compromised. If the address is instead immutable, a compromised owner address may do less damage to the protocol or allow users more time to exit the protocol.

Also, the owner of the NRN contract can grant roles- for example, the minter role. In case of a compromised owner, an attacker could mint an arbitrary amount of NRN tokens. It's safer to immutably grant minting permission to the contracts/EOAs that need it. Using a multisig wallet would reduce the centralization risk of an EOA with minting permission.

There is also centralization risk regarding the game server and battle result submission; the protocol could manipulate these results for their own gain. However, this risk is to be expected.

## Summary/Conclusion
There are a few concerns that I have regarding the game balance of the protocol. The centralization risks can be mitigated to a large degree.

I found the sqrt `stakingFactor` calculation to be a very nice mechanism, and other than the main area of systemic concern, I didn't spot any 'problem areas' in the codebase's architecture, which is very good. Most of the H/M issues I found are easily fixed and do not stem from deep design problems.

### Time spent:
45 hours