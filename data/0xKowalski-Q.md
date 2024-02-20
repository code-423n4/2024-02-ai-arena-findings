[L-1] Potential DoS from High `roundId`/`winnerAddresses` in [`MergingPool::claimRewards`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L134-L167)

With increased round counts and/or `winnerAddresses`, users might face a DoS scenario when claiming large amounts of overdue rewards, due to the nested loops.

```
   uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
                    claimIndex += 1;
                }
            }
        }
```

Consider a user who has participated in 100 bi-weekly rounds over four years without claiming any rewards. Over this period, the platform's user base—and consequently, the number of `winnerAddresses`—has grown significantly. Initially, the user could have easily claimed rewards when the number of winners per round was low. However, as the platform expanded, each round's `winnerAddresses` array became substantially larger, due to admins increasing the size to account for new players.

Now imagine the user attempts to claim rewards from these 100 rounds. The transaction has to navigate through the much larger `winnerAddresses` for each round. This translates to higher gas costs, which, assuming the user mints in some of these rounds, might surpass the block's gas limit. As a result, the user could find themselves unable to claim their rewards.

Due to it being unlikely a user actually mints enough times in the for loop to accumulate significant gas usage, the severity of this scenario is low, but not impossible, so my recommendation to mitigate it, is to introduce a `claimReward` method, which allows claims for a single `roundId`, reducing gas usage and preventing a complete loss in the ability to claim rewards.
