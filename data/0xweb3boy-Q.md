[QA-1] - ClaimNRN will revert when everybody send their 100% points to the Merging Pool

```solidity
function claimNRN() external {
        require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
        uint256 claimableNRN = 0;
        uint256 nrnDistribution;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound]; 
            numRoundsClaimed[msg.sender] += 1;
        }
        if (claimableNRN > 0) {
            amountClaimed[msg.sender] += claimableNRN;
            _neuronInstance.mint(msg.sender, claimableNRN);
            emit Claimed(msg.sender, claimableNRN);
        }
    }
```

The above function will revert when everyone decide to send their 100% points to the MergingPool since `mergingPortion`  matters in `_addResultPoints`

```solidity
if (battleResult == 0) {
            /// If the user won the match

            /// If the user has no NRNs at risk, then they can earn points
            if (stakeAtRisk == 0) {
                points = stakingFactor[tokenId] * eloFactor;
            }

            /// Divert a portion of the points to the merging pool
            uint256 mergingPoints = (points * mergingPortion) / 100;
            points -= mergingPoints;
            _mergingPoolInstance.addPoints(tokenId, mergingPoints);
``` 

If user won a match and his mergingPortion is 100%, all of his points are sent to mergingPool.And the line `points -= mergingPoints`, points will be 0 in that case and no points are added to the rankedBattle accounting here:

```solidity
accumulatedPointsPerFighter[tokenId][roundId] += points;
            accumulatedPointsPerAddress[fighterOwner][roundId] += points;
            totalAccumulatedPoints[roundId] += points;
```
and `totalAccumulatedPoints[roundId] += points` will be 0 in that case and so in `claimNRN()` function will revert since there will be a divison by 0

```solidity
claimableNRN +=(accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution) / totalAccumulatedPoints[currentRound]
```
