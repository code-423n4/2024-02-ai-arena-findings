## When making new round, consider to transfer all NRN from `StakeAtRisk.sol` to avoid locked NRN in the contract

this allow to unintended/rogue direct NRN transfer to StakeAtRisk contract to be rescued

Github: [143](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L143)

### Proof of Concept:
1. User make unintended NRN transfer directly to contract
2. Round complete and RankedBattle contract calling `setNewRound` function that call the `_sweepLostStake` function
3. `_sweepLostStake` only taking the amount of NRN from `totalStakeAtRisk[roundId]`, meaning only taking the current roundId lostStake, then increment the roundId
4. NRN from step 1 is locked forever

StakeAtRisk.sol
```
    function setNewRound(uint256 roundId_) external {
        require(msg.sender == _rankedBattleAddress, "Not authorized to set new round");
        bool success = _sweepLostStake();
        if (success) {
            roundId = roundId_;
        }
    }
.
.
.
    function _sweepLostStake() private returns(bool) {
@>      return _neuronInstance.transfer(treasuryAddress, totalStakeAtRisk[roundId]);
    }
```

### Mitigation
Consider to replace the amount of transferred NRN from `_sweepLostStake` function to total amount of balance held by StakeAtRisk contract instead only from the latest round completed:

```diff
    function _sweepLostStake() private returns(bool) {
        uint256 totalBalance = _neuronInstance.balanceOf(address(this));
-       return _neuronInstance.transfer(treasuryAddress, totalStakeAtRisk[roundId]);
+       return _neuronInstance.transfer(treasuryAddress, totalBalance);
    }
```