[L-01] Wrong accounting of ``totalBattles``

```solidity
    /// @notice Number of total battles.
    uint256 public totalBattles = 0;
```

``totalBattles`` is increased each time [updateBattleRecord()](), the problem is that this will be called for each user, with the user specific result. This leads to a wrong accounting, as the ``totalBattles`` variable will represent double the count of the actual battles that took place. Consider modifiying the code as follows:
```diff
File: src/RankedBattle.sol

    function updateBattleRecord(
        uint256 tokenId, 
        uint256 mergingPortion,
        uint8 battleResult,
        uint256 eloFactor,
        bool initiatorBool
    ) 
        external 
    { 
        ...
        if (initiatorBool) {
            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
+           totalBattles += 1;
        }
        /// INFO: This will be called for each user, not keeping correct track.
-        totalBattles += 1;
    }
```
[L-02] Last Wei of Neuron token can't be minted
```diff
File: src/Neuron.sol
    function mint(address to, uint256 amount) public virtual {
-        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
+        require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```
[L-03] getFighterPoints() returns the wrong value
The ``getFighterPoints()`` function is expected to return an array with the points of each fighter until a certain token Id is reached. The function will return only 1 element as the length of the array is set to 1. Consider the following fix: 

```diff
File: src/MergingPool.sol
    function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
-       uint256[] memory points = new uint256[](1);
+       uint256[] memory points = new uint256[](maxId);
        for (uint256 i = 0; i < maxId; i++) {
            points[i] = fighterPoints[i];
        }
        return points;
    }

```
[L-04] pickWinner() there is no check if the selected winner can mint more NFTs, there is a maximum of 10 NFTs per account
Consider adding such a check before you select a winner or add the possibility for a winner to chose another address to which the NFT he won can be minted.

```diff
File: src/MergingPool.sol
    function pickWinner(uint256[] calldata winners) external {
        require(isAdmin[msg.sender]);
        require(winners.length == winnersPerPeriod, "Incorrect number of winners");
        require(!isSelectionComplete[roundId], "Winners are already selected");
        uint256 winnersLength = winners.length;
        address[] memory currentWinnerAddresses = new address[](winnersLength);
        for (uint256 i = 0; i < winnersLength; i++) {
            /// INFO: There is no check to see if the winner don't already have the max number of NFTs per account
            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
            totalPoints -= fighterPoints[winners[i]];
            fighterPoints[winners[i]] = 0;
        }
        winnerAddresses[roundId] = currentWinnerAddresses;
        isSelectionComplete[roundId] = true;
        roundId += 1;
    }
```
[L-05] globalStakedAmount is not updated when user has a part of his staked NRN put at risk 
Consider the following fix, also if a stake at risk is reclaimed by the user the reclaimed amount would have to be added to the globalStakedAmount
```diff
    function _addResultPoints(
        uint8 battleResult, 
        uint256 tokenId, 
        uint256 eloFactor, 
        uint256 mergingPortion,
        address fighterOwner
    ) 
        private 
    {
        ...
        if (success) {
           /// INFO: seems like globalStakedAmount is not updated
           _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
           amountStaked[tokenId] -= curStakeAtRisk;
+          globalStakedAmount -= globalStakedAmount;
        }

    }
```
