| |Issue|Instances| Gas Savings
|-|:-|:-:|:-:|
| [[G-01](#g-01)] | The burnFrom function evaluates the value of allowance(account, msg.sender) multiple times | 1| 237|
| [[G-02](#g-02)] | The claimNRN function uses the value of numRoundsClaimed[msg.sender] multiple times | 1| 344|
| [[G-03](#g-03)] | The updateBattleRecord function uses 100 several times instead of setting a constant | 1| |

### [G-01] The burnFrom function evaluates the value of allowance(account, msg.sender) multiple times
The burnFrom function evaluates the value of allowance(account, msg.sender) multiple times. It can be replaced by uint256 allow to avoid double calculations

```solidity
File: src/Neuron.sol


    function burnFrom(address account, uint256 amount) public virtual {
+       uint256 allow = allowance(account, msg.sender);
+       require(allow >= amount,"ERC20: burn amount exceeds allowance");

-       require(
-           allowance(account, msg.sender) >= amount, 
-           "ERC20: burn amount exceeds allowance"
-       );
-       uint256 decreasedAllowance = allowance(account, msg.sender) - amount;

+       uint256 decreasedAllowance = allow - amount;
        _burn(account, amount);
        _approve(account, msg.sender, decreasedAllowance);
    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L196-L204

Before making changes
[PASS] testBurnFrom() (gas: 103644)

After making changes
[PASS] testBurnFrom() (gas: 103407)

### [G-02] The claimNRN function uses the value of numRoundsClaimed[msg.sender] multiple times
The claimNRN function uses the value of numRoundsClaimed[msg.sender] multiple times.It can be replaced by uint32 _numRoundsClaimed to avoid double calculations.
```solidity
File: src/RankedBattle.sol


    function claimNRN() external {
-       require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
+       uint32 _numRoundsClaimed = numRoundsClaimed[msg.sender];
+       require(_numRoundsClaimed < roundId, "Already claimed NRNs for this period");
        uint256 claimableNRN = 0;
        uint256 nrnDistribution;
+       uint32 lowerBound = _numRoundsClaimed;
-       uint32 lowerBound = numRoundsClaimed[msg.sender];
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
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L294-L311

Before making changes
[PASS] testClaimNRN() (gas: 1781228)

After making changes
[PASS] testClaimNRN() (gas: 1780884)

### [G-03] The updateBattleRecord function uses 100 several times instead of setting a constant

```solidity
File: src/RankedBattle.sol
+       uint256 public constant x = 100;
+       require(mergingPortion <= x);
-       require(mergingPortion <= 100);
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L332
```

```solidity
File: src/RankedBattle.sol
+       uint256 mergingPoints = (points * mergingPortion) / x;
-       uint256 mergingPoints = (points * mergingPortion) / 100;
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L449
```
Before making changes
[PASS] testSetNewRoundFromAdmin() (gas: 957090)
[PASS] testClaimNRN() (gas: 1781228)
[PASS] testGetBattleRecord() (gas: 874511)
[PASS] testGetCurrentStakingData() (gas: 876469)
[PASS] testgetUnclaimedNRN() (gas: 963071)
[PASS] testUpdateBattleRecordPlayerLossBattle() (gas: 854453)
[PASS] testUpdateBattleRecordPlayerTiedBattle() (gas: 754196)
[PASS] testUpdateBattleRecordPlayerWonBattle() (gas: 958706)

After making changes
[PASS] testSetNewRoundFromAdmin() (gas: 957067)
[PASS] testClaimNRN() (gas: 1781226)
[PASS] testGetBattleRecord() (gas: 874488)
[PASS] testGetCurrentStakingData() (gas: 876424)
[PASS] testgetUnclaimedNRN() (gas: 963070)
[PASS] testUpdateBattleRecordPlayerLossBattle() (gas: 854430)
[PASS] testUpdateBattleRecordPlayerTiedBattle() (gas: 754173)
[PASS] testUpdateBattleRecordPlayerWonBattle() (gas: 958705).
