https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L139-L167

## Impact

I was thinking that perhaps `claimRewards` could potentially generate issues for users who win rewards in a fairly **advanced roundId**, as claimRewards starts with a `lowerBound == 0` for new users. Additionally, it contains a loop that checks if `msg.sender == winnerAddresses[currentRound][j]`. Therefore, I created a test to understand how problematic this function can be:

## POC

```
        function testClaimRewardsAdvancedRoundIdWithNewUser() public {

        address winner = makeAddr("winner");
        address player3 = makeAddr("player3");
        address player4 = makeAddr("player4");
        address player5 = makeAddr("player5");
        address player6 = makeAddr("player6");
        address player7 = makeAddr("player7");

        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_DELEGATED_ADDRESS);
        _mintFromMergingPool(winner);
        _mintFromMergingPool(player3);
        _mintFromMergingPool(player4);
        _mintFromMergingPool(player5);
        _mintFromMergingPool(player6);
        _mintFromMergingPool(player7);

        _mergingPoolContract.updateWinnersPerPeriod(7);

        string[] memory _modelURIs = new string[](2);
        _modelURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelURIs[1] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        string[] memory _modelTypes = new string[](2);
        _modelTypes[0] = "original";
        _modelTypes[1] = "original";
        uint256[2][] memory _customAttributes = new uint256[2][](2);
        _customAttributes[0][0] = uint256(1);
        _customAttributes[0][1] = uint256(80);
        _customAttributes[1][0] = uint256(1);
        _customAttributes[1][1] = uint256(80);

        uint256[] memory _winners = new uint256[](7);

        _winners[0] = 0;
        _winners[1] = 1;
        _winners[2] = 2;
        _winners[3] = 3;
        _winners[4] = 4;
        _winners[5] = 5;
        _winners[6] = 6;

        uint256[] memory _previousWinners = new uint256[](7);

        _previousWinners[0] = 0;
        _previousWinners[1] = 1;
        _previousWinners[2] = 3;
        _previousWinners[3] = 4;
        _previousWinners[4] = 5;
        _previousWinners[5] = 6;
        _previousWinners[6] = 7;

        for(uint256 i = 0; i < 6000; i++){
            _mergingPoolContract.pickWinner(_previousWinners);
        }

        _mergingPoolContract.pickWinner(_winners);

        uint256 preGas = gasleft();

        vm.startPrank(winner);

        _mergingPoolContract.claimRewards(_modelURIs, _modelTypes, _customAttributes);

        vm.stopPrank();

        uint256 postGas = gasleft();

        console.log("gas used:");
        console.log(preGas - postGas);

    }
```

Increasing to 7 winners per round and with 6000 roundIds previously played, approximately when a transaction reaches the 30M limit for new users in the protocol, considering that each roundId is 14 days, we would not have problems with the current implementation for new users starting at elevated roundIds.