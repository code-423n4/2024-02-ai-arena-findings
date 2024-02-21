
## [L-1] The NRN rewards are a time bomb

When the users claim rewards as NRN tokens the tokens are minted, but when they lose tokens the tokens are sent to treasury contract. Since the number of possible minted tokens are locked the minting will not be possible at some point.
To be more precise from the deployment the contract has a max supply of `1e9` tokens (omitting the decimals). Then it mints `2e8` and `5e8` to the treasury and contributors. And left the `3e8` as a reward. 
Currently, each round will be distributed `5000` tokens and each round is longs about 1 week. By doing simple calculating we understand that it's possible to run a maximum `60000` rounds. It sounds a lot, but the length of the round and the number of distributed tokens is changeable. For example, to incentivize players and attract new ones the number of tokens can significantly increase and the duration of the round decrease.
I think the rewards should be distributed from the treasury contract, otherwise, it may break someday. Such an approach will guarantee an unbreakable loop.

```solidity
            _neuronInstance.mint(msg.sender, claimableNRN);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L308

```solidity
    function setRankedNrnDistribution(uint256 newDistribution) external {
        require(isAdmin[msg.sender]);
        rankedNrnDistribution[roundId] = newDistribution * 10**18;
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L218-L221

## [L-2] Inconsistencies with the replenish time

Currently, the user can use the battery even after the replenishment time has passed. This is because the values will not be updated automatically and the user should interact with the `spendVoltage` function or start a fight.
However, within this period the user can use the battery. This will not give any profit and just wastes the battery. Consider adding a check in the `useVoltageBattery()` function or revert in this case.
```solidity

```

```diff
    function useVoltageBattery() public {
        require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
+       if (ownerVoltageReplenishTime[msg.sender] <= block.timestamp) {
+           _replenishVoltage(msg.sender);
+            return;
+        }
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L93-L99

## [NC-1] Add proper checks

The `voltageSpent` number can't be more than `100`, otherwise the ownerVoltage[spender] will overflow

```solidity
    function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }
        ownerVoltage[spender] -= voltageSpent;
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L105-L112


## [NC-2] Give `isOwner` privileges immediately

The `transferOwnership` function changes the main owner of the contract. However, the `isAdmin` mapping is not updated properly. The `isAdmin` should be taken from the previous owner and the new owner should be marked in the mapping.

```diff
    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
+       isAdmin[newOwnerAddress] = true;
+       isAdmin[msg.sender] = false;
    }
```
The issue persists between multiple contracts. One of them:
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L64-L67

##  [NC-3] Increase the reliability of the system

The rounds longs at least one week. Time delay between rounds can be added to increase the reliability of the system.

```solidity
    function setNewRound() external {
        require(isAdmin[msg.sender]);
        require(totalAccumulatedPoints[roundId] > 0);
        roundId += 1;
        _stakeAtRiskInstance.setNewRound(roundId);
        rankedNrnDistribution[roundId] = rankedNrnDistribution[roundId - 1];
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L233-L239

After selecting a number of winners for the current round, the function should be locked, until the next period.

```solidity
    function updateWinnersPerPeriod(uint256 newWinnersPerPeriodAmount) external {
        require(isAdmin[msg.sender]);
        winnersPerPeriod = newWinnersPerPeriodAmount;
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L106-L109
