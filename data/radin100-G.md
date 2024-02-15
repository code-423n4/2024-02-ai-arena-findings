### [G-1] 'FighterOps.sol::Fighter::generetion' type uint16 is gas cheaper
 Code snipped:
 https://github.com/code-423n4/2024-02-ai-arena/blob/70b73ce5acaf10bc331cf388d35e4edff88d4541/src/FighterOps.sol#L36-L46

 When run test in 'FighterFarm.t.sol': 

      function testGetAllFighterInfo() public {
        _mintFromMergingPool(_ownerAddress);
        uint256 gasUsed = gasleft();
        (, , , , , , uint16 generation) = _fighterFarmContract
            .getAllFighterInfo(0);
        console.log(
            "Gas used for getting AI generation: ",
            gasUsed - gasleft()
        );
    }

### [G-2] 'MergingPool.sol::pickWinner' provides unnecessary require:
Details:
isSelectionComplete[roundId] will always return false, because after this require there is nothing that can stop roundId to get incremented. And removing this require saves 295 gas to the caller

Code Snipped:
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L118-L132

Recommendation: 
Remove the unnecessary require to save gas as shown below

      function pickWinner(uint256[] calldata winners) external {
        require(isAdmin[msg.sender]);
        require(winners.length == winnersPerPeriod, "Incorrect number of winners");
        -require(!isSelectionComplete[roundId], "Winners are already selected");
        uint256 winnersLength = winners.length;
        address[] memory currentWinnerAddresses = new address[](winnersLength);
        for (uint256 i = 0; i < winnersLength; i++) {
            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
            totalPoints -= fighterPoints[winners[i]];
            fighterPoints[winners[i]] = 0;
        }
        winnerAddresses[roundId] = currentWinnerAddresses;
        isSelectionComplete[roundId] = true;
        roundId += 1;
    }
