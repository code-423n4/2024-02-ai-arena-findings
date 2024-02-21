# Gas Optimizations Report

## Note to the Judge:
I want to assure you that all gas optimizations in question were researched manually without the use of any automated tools or bots. I researched each of the gas issues below mannually to provide you with complete reports about each issue. Your understanding on this matter is greatly appreciated.

| ID     | Optimization                                                                                            |
|--------|---------------------------------------------------------------------------------------------------------|
| [G-01] | Redundant Code in the `constructor` of the `AiArenaHelper` contract                      |
| [G-02] | Use of Increment Operator in the `pickWinner` function of the `MergingPool` contract                                           |
| [G-03] | Optimizing `uint32` Declaration in the `claimRewards` function of the `MergingPool` contract                                                              |


## [G-01] Redundant Code in the `constructor` of the `AiArenaHelper` contract  


**Description**
In the provided Solidity code for the `AiArenaHelper` contract, there exists a redundant assignment within the constructor function. The line `attributeProbabilities[0][attributes[i]] = probabilities[i];` duplicates the task performed by the `addAttributeProbabilities` function.

**Code:**

```diff

constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // Initialize the probabilities for each attribute
    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
-       attributeProbabilities[0][attributes[i]] = probabilities[i]; // Redundant assignment
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
}
```

**Gas report before the optimization:**

| src/AiArenaHelper.sol:AiArenaHelper contract |                 |        |        |        |         |
|----------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                              | Deployment Size |        |        |        |         |
| 1741279                                      | 8445            |        |        |        |         |
| Function Name                                | min             | avg    | median | max    | # calls |
| addAttributeDivisor                          | 3612            | 26868  | 26868  | 50124  | 2       |
| addAttributeProbabilities                    | 129284          | 129284 | 129284 | 129284 | 1       |
| attributeToDnaDivisor                        | 921             | 1921   | 1921   | 2921   | 2       |
| createPhysicalAttributes                     | 1043            | 68269  | 83687  | 85463  | 53      |
| deleteAttributeProbabilities                 | 2536            | 34295  | 33895  | 66854  | 4       |
| dnaToIndex                                   | 9038            | 9038   | 9038   | 9038   | 1       |
| getAttributeProbabilities                    | 3698            | 6364   | 7698   | 7698   | 3       |
| transferOwnership                            | 2491            | 4027   | 4027   | 5564   | 2       |

**Gas report after the optimization:**

| src/AiArenaHelper.sol:AiArenaHelper contract |                 |        |        |        |         |
|----------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                              | Deployment Size |        |        |        |         |
| 1661623                                      | 8308            |        |        |        |         |
| Function Name                                | min             | avg    | median | max    | # calls |
| addAttributeDivisor                          | 3612            | 26868  | 26868  | 50124  | 2       |
| addAttributeProbabilities                    | 129284          | 129284 | 129284 | 129284 | 1       |
| attributeToDnaDivisor                        | 921             | 1921   | 1921   | 2921   | 2       |
| createPhysicalAttributes                     | 1043            | 68269  | 83687  | 85463  | 53      |
| deleteAttributeProbabilities                 | 2536            | 34295  | 33895  | 66854  | 4       |
| dnaToIndex                                   | 9038            | 9038   | 9038   | 9038   | 1       |
| getAttributeProbabilities                    | 3698            | 6364   | 7698   | 7698   | 3       |
| transferOwnership                            | 2491            | 4027   | 4027   | 5564   | 2       |


**You can see the difference in the gas usage**



## [G-02] Use of Increment Operator in the `pickWinner` function of the `MergingPool` contract


**Description**
In the provided smart contract function `pickWinner`, there is a gas optimization opportunity regarding the method used to increment the `roundId` variable. Currently, the code uses the `roundId += 1` statement to increment the `roundId` variable after selecting winners for the current round. However, replacing it with the `roundId++` increment operator can offer a slight gas optimization.

**Code:**

```diff

    function pickWinner(uint256[] calldata winners) external {
        require(isAdmin[msg.sender]);
        require(
            winners.length == winnersPerPeriod, // G? `winners.length` can be declared as a variable before the `require` statement?
            "Incorrect number of winners"
        );
        require(!isSelectionComplete[roundId], "Winners are already selected");
        uint256 winnersLength = winners.length;
        address[] memory currentWinnerAddresses = new address[](winnersLength);
        for (uint256 i = 0; i < winnersLength; i++) {
            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(
                winners[i]
            );
            totalPoints -= fighterPoints[winners[i]];
            fighterPoints[winners[i]] = 0;
        }
        winnerAddresses[roundId] = currentWinnerAddresses;
        isSelectionComplete[roundId] = true;
-       roundId += 1;
+       roundId++;
    }
```

**Gas report before the optimization:**

| src/MergingPool.sol:MergingPool contract |                 |        |        |        |         |
|------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                          | Deployment Size |        |        |        |         |
| 912202                                   | 4340            |        |        |        |         |
| Function Name                            | min             | avg    | median | max    | # calls |
| addPoints                                | 2603            | 41316  | 48274  | 48274  | 10      |
| adjustAdminAccess                        | 2537            | 16694  | 22773  | 24773  | 3       |
| claimRewards                             | 780715          | 788992 | 788992 | 797269 | 2       |
| getFighterPoints                         | 1029            | 2011   | 2011   | 2994   | 2       |
| getUnclaimedRewards                      | 783             | 3517   | 3517   | 6251   | 2       |
| isAdmin                                  | 582             | 1248   | 582    | 2582   | 3       |
| isSelectionComplete                      | 485             | 985    | 485    | 2485   | 4       |
| pickWinner                               | 4877            | 97087  | 112140 | 129090 | 6       |
| totalPoints                              | 385             | 1051   | 385    | 2385   | 3       |
| transferOwnership                        | 2498            | 4034   | 4034   | 5571   | 2       |
| updateWinnersPerPeriod                   | 2432            | 4935   | 4935   | 7439   | 2       |
| winnerAddresses                          | 749             | 749    | 749    | 749    | 6       |
| winnersPerPeriod                         | 317             | 1317   | 1317   | 2317   | 2       |

**Gas report after the optimization:**

| src/MergingPool.sol:MergingPool contract |                 |        |        |        |         |
|------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                          | Deployment Size |        |        |        |         |
| 911602                                   | 4337            |        |        |        |         |
| Function Name                            | min             | avg    | median | max    | # calls |
| addPoints                                | 2603            | 41316  | 48274  | 48274  | 10      |
| adjustAdminAccess                        | 2537            | 16694  | 22773  | 24773  | 3       |
| claimRewards                             | 780715          | 788992 | 788992 | 797269 | 2       |
| getFighterPoints                         | 1029            | 2011   | 2011   | 2994   | 2       |
| getUnclaimedRewards                      | 783             | 3517   | 3517   | 6251   | 2       |
| isAdmin                                  | 582             | 1248   | 582    | 2582   | 3       |
| isSelectionComplete                      | 485             | 985    | 485    | 2485   | 4       |
| pickWinner                               | 4877            | 97077  | 112128 | 129078 | 6       |
| totalPoints                              | 385             | 1051   | 385    | 2385   | 3       |
| transferOwnership                        | 2498            | 4034   | 4034   | 5571   | 2       |
| updateWinnersPerPeriod                   | 2432            | 4935   | 4935   | 7439   | 2       |
| winnerAddresses                          | 749             | 749    | 749    | 749    | 6       |
| winnersPerPeriod                         | 317             | 1317   | 1317   | 2317   | 2       |

**You can see the difference in the gas usage**




## [G-03] Optimizing `uint32` Declaration in the `claimRewards` function of the `MergingPool` contract


**Description**
The `claimRewards` function in the provided code contains a gas optimization opportunity by directly initializing the `uint32` variable `currentRound` within the for loop declaration instead of pre-declaring and initializing it outside the loop. This optimization reduces redundant variable declarations and enhances code readability.


**Code:**

```diff

    function claimRewards(
        string[] calldata modelURIs,
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) external {
        uint256 winnersLength;
        uint32 claimIndex = 0;
-       uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (
-           uint32 currentRound = lowerBound;
+           uint32 currentRound = numRoundsClaimed[msg.sender];
            currentRound < roundId;
            currentRound++
        ) {
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

***Link To Code:***
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L148-L149

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L175-L176

**Gas report before the optimization:**

| src/MergingPool.sol:MergingPool contract |                 |        |        |        |         |
|------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                          | Deployment Size |        |        |        |         |
| 912202                                   | 4340            |        |        |        |         |
| Function Name                            | min             | avg    | median | max    | # calls |
| addPoints                                | 2603            | 41316  | 48274  | 48274  | 10      |
| adjustAdminAccess                        | 2537            | 16694  | 22773  | 24773  | 3       |
| claimRewards                             | 780715          | 788992 | 788992 | 797269 | 2       |
| getFighterPoints                         | 1029            | 2011   | 2011   | 2994   | 2       |
| getUnclaimedRewards                      | 783             | 3517   | 3517   | 6251   | 2       |
| isAdmin                                  | 582             | 1248   | 582    | 2582   | 3       |
| isSelectionComplete                      | 485             | 985    | 485    | 2485   | 4       |
| pickWinner                               | 4877            | 97087  | 112140 | 129090 | 6       |
| totalPoints                              | 385             | 1051   | 385    | 2385   | 3       |
| transferOwnership                        | 2498            | 4034   | 4034   | 5571   | 2       |
| updateWinnersPerPeriod                   | 2432            | 4935   | 4935   | 7439   | 2       |
| winnerAddresses                          | 749             | 749    | 749    | 749    | 6       |
| winnersPerPeriod                         | 317             | 1317   | 1317   | 2317   | 2       |

**Gas report after the optimization:**

| src/MergingPool.sol:MergingPool contract |                 |        |        |        |         |
|------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                          | Deployment Size |        |        |        |         |
| 911202                                   | 4335            |        |        |        |         |
| Function Name                            | min             | avg    | median | max    | # calls |
| addPoints                                | 2603            | 41316  | 48274  | 48274  | 10      |
| adjustAdminAccess                        | 2537            | 16694  | 22773  | 24773  | 3       |
| claimRewards                             | 780710          | 788987 | 788987 | 797264 | 2       |
| getFighterPoints                         | 1029            | 2011   | 2011   | 2994   | 2       |
| getUnclaimedRewards                      | 775             | 3509   | 3509   | 6243   | 2       |
| isAdmin                                  | 582             | 1248   | 582    | 2582   | 3       |
| isSelectionComplete                      | 485             | 985    | 485    | 2485   | 4       |
| pickWinner                               | 4877            | 97087  | 112140 | 129090 | 6       |
| totalPoints                              | 385             | 1051   | 385    | 2385   | 3       |
| transferOwnership                        | 2498            | 4034   | 4034   | 5571   | 2       |
| updateWinnersPerPeriod                   | 2432            | 4935   | 4935   | 7439   | 2       |
| winnerAddresses                          | 749             | 749    | 749    | 749    | 6       |
| winnersPerPeriod                         | 317             | 1317   | 1317   | 2317   | 2       |

**You can see the difference in the gas usage**



