## [L-01] `globalStakedAmount` is not updated when users loss stake

In `RankedBattle` contract `globalStakedAmount` is used to keep track of the amount of stake that is staked in the system currently by all of the users.

But when the rounds ends stake lost by all of the users will be moved to treasury.
So while incrementing round `globalStakedAmount` should be reduced by `_stakeAtRiskInstance.totalStakeAtRisk(roundId)`.


## [L-02] Incorrect fighter balance limit check
In `FighterFarm` contract while checking that the fighter NFT balance of user isn't exceeded MAX_FIGHTERS_ALLOWED value `<` is used instead of using `<=`

* [FighterFarm.sol#L542](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L542)
* [FighterFarm.sol#L495](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L495)

Due to this incorrect check a user can't own 10 NFTs he can only own 9 NFTs

## [L-03] DEFAULT_ADMIN_ROLE not assigned 

In Neuron ERC20 contract DEFAULT_ADMIN_ROLE is not assigned to any user. Due to this the role once assigned cannot be revoked.

So assign DEFAULT_ADMIN_ROLE role to _ownerAddress.




