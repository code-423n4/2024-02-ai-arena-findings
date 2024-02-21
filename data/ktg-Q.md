### L-01:  MergingPool.claimRewards may DOS users
Function MergingPoo.claimRewards is currently implemented as:
```solidity
unction claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
        external 
    {
        uint256 winnersLength;
        uint32 claimIndex = 0;
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
        if (claimIndex > 0) {
            emit Claimed(msg.sender, claimIndex);
        }
    }
```

The function will try to claim all reward from `numRoundsClaimed` to the last roundId. If a player never claim reward before, `numRoundsClaimed` = 0. If there's 1 round each week and the battle arena has run for 1 year (52 weeks), 5 winners each round, then the function must loop 260 times and this can DOS user.
