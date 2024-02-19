# G-1) If `numRoundsClaimed[msg.sender] == 0` it's unnecessary to calculate `claimableNRN`
If a player is newly registered or has stopped playing for a long time, there will be lots of 0 value in `numRoundsClaimed`. For each 0 value there is unnecessary `claimableNRN` [calculation](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L299-L305). Similar issue also can be found [here](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L390-L395)

Also `numRoundsClaimed[msg.sender]` should be cached. (This is mentioned in bot races but I wrote it in recommendation to make it clear)
## Recommended Mitigation Steps
```diff
diff --git a/src/RankedBattle.sol#L299-L305
        uint32 lowerBound = numRoundsClaimed[msg.sender];
+       uint256 accumulatedPoints;
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
+           accumulatedPoints = accumulatedPointsPerAddress[msg.sender][currentRound];
            if (accumulatedPoints == 0){
                lowerBound += 1;
            }
+           else{
                nrnDistribution = getNrnDistribution(currentRound);
                claimableNRN += (
-                   accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution
+                   accumulatedPoints * nrnDistribution  
                ) / totalAccumulatedPoints[currentRound];
-               numRoundsClaimed[msg.sender] += 1;
+               lowerBound += 1;
+           }
        }
+       numRoundsClaimed[msg.sender] = lowerBound;

```

# G-2) Same thing done twice in constructor
[This line](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L49) run already in `addAttributeProbabilities` function. 
## Recommended Mitigation Steps
```diff
diff --git a/src/AiArenaHelper.sol#L44-L51
        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
-           attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
```