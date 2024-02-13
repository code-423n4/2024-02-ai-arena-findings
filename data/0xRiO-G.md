# Gas Analysis Report

# [G-1] AiArenaHelper::attributes.length can be initialized as a constant

## Description
The `attributes.length` variable is used throughout the contract and can be initialized as constant. This constant declaration oversight can lead to increased gas costs during contract deployment and function executions. Also array attributes is not changing throught the contract

## Cause
The cause of the issue is the absence of a constant declaration for the `attributes.length` variable. As a result, each reference to `attributes.length` incurs gas costs for storage reads, impacting contract efficiency.


### Proposed Correction:
```solidity
uint8 constant public attributesLength = 6;
```
# [G-2] FighterFarm::redeemMintPass require statement can be made more gas efficient 

## Description
The `redeemMintPass` function contains redundant require statements to validate the lengths of input arrays, leading to inefficient gas usage.

## Cause
The cause of the issue is the redundant require statements used to validate the lengths of input arrays (`mintpassIdsToBurn`, `mintPassDnas`, `fighterTypes`, `modelHashes`, and `modelTypes`). These redundant require statements contribute to gas inefficiency.

### Proposed Correction:
- Making a variable length and then comparing with 
```solidity
// Validate lengths of input arrays
uint256 length = mintpassIdsToBurn.length;
require(
    length == mintPassDnas.length && 
    length == fighterTypes.length && 
    length == modelHashes.length &&
    length == modelTypes.length,
    "Array lengths do not match"
);

for (uint16 i = 0; i < length; i++) {
    require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]), "Sender is not the owner of the mint pass");
    _mintpassInstance.burn(mintpassIdsToBurn[i]);
    _createNewFighter(
        msg.sender, 
        uint256(keccak256(abi.encode(mintPassDnas[i]))), 
        modelHashes[i], 
        modelTypes[i],
        fighterTypes[i],
        iconsTypes[i],
        [uint256(100), uint256(100)]
    );
}
```

# [G-3] Using `!=` instead of `<` or `>` in loops/conditions will gave gas
## PoC
- This is a function with loop that does nothing execution cost 212 gas when using `i>0` and 206 when using `i!=0` with increasing iteration this difference can quickly increase resulting in high gas cost for calling functions. 
```
function checkingGasOfLoop() public {
        for (uint256 i =1;i!=0;){
        unchecked{
            --i;
        }
     }
    }
```
- One more example where we can save gas
```
if (points > 0) {}
```

## Instances
Total 22 instance throughout the scope
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L48
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L73
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L99
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L136
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L148
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L178
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L211
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L249
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L124
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L149
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L152
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L176
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L178
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L207
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L131
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L299
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L390
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L469
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L488
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L164
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L245
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L306

# [G-4] Updating `numRoundsClaimed[msg.sender]` mapping in RankedBattle::claimNRN function can be made gas efficent.

## Description
The `claimNRN` function contains inefficiencies in updating the `numRoundsClaimed` mapping, leading to unnecessary gas consumption.

### Proposed Correction:
```diff
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
-       numRoundsClaimed[msg.sender] += 1;
    }
    // Update numRoundsClaimed mapping outside the loop
+    numRoundsClaimed[msg.sender] = uint32(roundId-1); 
    if (claimableNRN > 0) {
        amountClaimed[msg.sender] += claimableNRN;
        _neuronInstance.mint(msg.sender, claimableNRN);
        emit Claimed(msg.sender, claimableNRN);
    }
}
```
## Gas difference 
- Before Function gas cost : 58_844
- After updating it outside loop Function gas cost : 38_860
## Instance
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L299-L305
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L149-L163