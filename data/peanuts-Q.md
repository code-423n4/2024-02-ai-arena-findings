### [L-01] The rounds in RankedBattle and MergingPool is not synced

In MergingPool.sol, `pickWinner()` can be called by the admin. Once the function runs, roundId is increased by 1. This roundId is not checked with the RankedBattle.

```
    function pickWinner(uint256[] calldata winners) external {
        require(isAdmin[msg.sender]);
        require(winners.length == winnersPerPeriod, "Incorrect number of winners");
        require(!isSelectionComplete[roundId], "Winners are already selected");
        uint256 winnersLength = winners.length;
        address[] memory currentWinnerAddresses = new address[](winnersLength);
        for (uint256 i = 0; i < winnersLength; i++) {
            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
            totalPoints -= fighterPoints[winners[i]];
            fighterPoints[winners[i]] = 0;
        }
        winnerAddresses[roundId] = currentWinnerAddresses;
        isSelectionComplete[roundId] = true;
>       roundId += 1;
    }
```

The roundId should be unanimous. Right now, the admin is able to call `pickWinner()` ahead of the RankedBattle roundID and make the roundId off synced.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L118-L132

### [L-02] updateWinnersPerPeriod in MergingPool.sol has no lower and upper bound

The admin can set the number of winners per period with `updateWinnersPerPeriod()`

```
    function updateWinnersPerPeriod(uint256 newWinnersPerPeriodAmount) external {
        require(isAdmin[msg.sender]);
        winnersPerPeriod = newWinnersPerPeriodAmount;
    }
```

There are no upper or lower bound to setting `winnersPerPeriod`. There can be zero winners per round or 10000 winners per round.

Not sure if it is intended to be able to set 0 winners per period. If not, best is to have a 0 value check in the function.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L106-L109

### [L-03] Use voltage battery assumes that voltage battery is id 0

Users can use a voltage battery to refresh their voltage. They will burn an ERC1155 token

```
    function useVoltageBattery() public {
        require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
>       _gameItemsContractInstance.burn(msg.sender, 0, 1);
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }
```

This function does not check that the voltage battery is indeed id 0. When the admin creates a game item, the token id will be increased monotonically.

```
    function createGameItem(
        string memory name_,
        string memory tokenURI,
        bool finiteSupply,
        bool transferable,
        uint256 itemsRemaining,
        uint256 itemPrice,
        uint16 dailyAllowance
    )
        public
    {
        require(isAdmin[msg.sender]);
        allGameItemAttributes.push(
            GameItemAttributes(
                name_,
                finiteSupply,
                transferable,
                itemsRemaining,
                itemPrice,
                dailyAllowance
            )
        );
        if (!transferable) {
          emit Locked(_itemCount);
        }
        setTokenURI(_itemCount, tokenURI);
>       _itemCount += 1;
    }
```

If the battery is not the first game item created, then the two contracts will not sync together. The user may burn another game item to recharge his voltage.

Check that the token being burned when recharging the voltage is indeed the battery, by checking the name of the ERC1155 token.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol

### [L-04] setBpsLostPerLoss can be set at a number greater than 10,000

The admin can use the function `setBpsLostPerLoss()` to set the bps

```
    function setBpsLostPerLoss(uint256 bpsLostPerLoss_) external {
        require(isAdmin[msg.sender]);
        bpsLostPerLoss = bpsLostPerLoss_;
    }
```

The default is 10, which is 0.1%, but this can be set to a number higher than 100%, which will affect the `curStakeAtRisk`.

```
        /// Potential amount of NRNs to put at risk or retrieve from the stake-at-risk contract
>       curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
        if (battleResult == 0) {
            /// If the user won the match
```

Limit the `setBpsLostPerLoss()` to a low maximum number, eg

```
    function setBpsLostPerLoss(uint256 bpsLostPerLoss_) external {
+       require(bpsLostPerLoss_ < 2000, "bps lost cannot be higher than 20%);
        require(isAdmin[msg.sender]);
        bpsLostPerLoss = bpsLostPerLoss_;
    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L226

### [L-05] setBpsLostPerLoss can be changed within the round, which may cause unfairness.

`bpsLostPerLoss` is used to calculate the `curStakeAtRisk`. If `bpsLostPerLoss` is set at 1000 (10%) at the start of the roundId, and changed to 10 (0.1%), those who lost the round will find it extremely hard to reclaim their NRN.

```
        curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
```

To ensure fairness, make sure `setBpsLostPerLoss()` can only be changed for the upcoming rounds, and not the current round.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L439

### [L-06] The game will eventually reach a point in time where no NRN can be awarded anymore

There is a max supply of 1 billion NRN, of which 700 million is minted at the start.

The ranked battle will mint and distribute tokens every time a round ends.

```
        if (claimableNRN > 0) {
            amountClaimed[msg.sender] += claimableNRN;
>           _neuronInstance.mint(msg.sender, claimableNRN);
            emit Claimed(msg.sender, claimableNRN);
```

In the constructor, the distribution is set at 5000 per round and can be changed at any time. (I feel that 5000 NRN is not enough to incentivise users to play a round, and so the number 5000 would be much higher)

```
        rankedNrnDistribution[0] = 5000 * 10**18;
```

```
    function setRankedNrnDistribution(uint256 newDistribution) external {
        require(isAdmin[msg.sender]);
        rankedNrnDistribution[roundId] = newDistribution * 10**18;
    }
```

There will be other game modes that will probably also mint more NRNs and also mint more tokens to giveaway. At a far point in time, the 300 million tokens remaining will be minted fully. `mint()` will always revert and the game modes will not have any more tokens minted.

```
    function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```

Consider being able to change the MAX_SUPPLY to account for the far future

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L155-L159

### [L-07] User can still earn points without staking NRNs.

Document notes:

> If a player does not stake NRNs, their NFT will not accrue Points or NRN rewards (i.e. Staking Factor would be 0 and the product of the equation is 0). 

It is possible that a player accrue Points without staking any NRNs. This issue assumes that `stake NRNs` means the process of staking NRNs only, and not having NRNs that are at risk.

```
    function _getStakingFactor(
        uint256 tokenId, 
        uint256 stakeAtRisk
    ) 
        private 
        view 
        returns (uint256) 
    {
      uint256 stakingFactor_ = FixedPointMathLib.sqrt(
>         (amountStaked[tokenId] + stakeAtRisk) / 10**18
      );
>     if (stakingFactor_ == 0) {
>       stakingFactor_ = 1;
      }
      return stakingFactor_;
    }    
```

Since `_getStakingFactor()` takes `stakeAtRisk` as well, and rounds up the `stakingFactor` to 1 should it be zero, users can simply get a fighter with some `stakeAtRisk` tokens, and withdraw all their current `amountStaked` tokens. 

This way, the user will not need to stake anything but will be able to earn points from their fighter.  

Make sure `_addResultPoints()` is called only if  `amountStaked[tokenId] > 0`, and not `amountStaked[tokenId] + stakeAtRisk > 0`

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L519-L534

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L342-L345

### [L-08] A fighter with `stakeAtRisk` tokens cannot redeem their tokens if the owner is changed

Let's say that there is a fighter with a lot of `stakeAtRisk` tokens. The owner of the fighter realizes that he does not want to play with this fighter anymore, so he unstakes all the tokens from the fighter, leaving only the `stakeAtRisk` tokens. Sicne there is no more `amountStaked`, the fighter can now be transferred. The owner can decide to sell his fighter to another person, and if the owner is malicious, he can upsell his fighter, saying that the fighter has a lot of `stakeAtRisk` that can potentially be claimed.

Now, if an unsuspecting buyer decides to buy the fighter in order to get the `stakeAtRisk` tokens, he will soon realize that he cannot withdraw any `stakeAtRisk` tokens even if he wins the battle because `_addResultPoints()` will be reverted. The reason behind this revert is due to the `reclaimNRN()` function.

In the `reclaimNRN()` function, there is this line `amountLost[fighterOwner] -= nrnToReclaim`. Since the fighterOwner has changed, the amountLost[fighterOwner] will now be 0. Winning a battle will result in this calculation underflowing, which will result in an error.

```
    function reclaimNRN(uint256 nrnToReclaim, uint256 fighterId, address fighterOwner) external {
        require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");
        require(
            stakeAtRisk[roundId][fighterId] >= nrnToReclaim, 
            "Fighter does not have enough stake at risk"
        );


        bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
        if (success) {
            stakeAtRisk[roundId][fighterId] -= nrnToReclaim;
            totalStakeAtRisk[roundId] -= nrnToReclaim;
>           amountLost[fighterOwner] -= nrnToReclaim;
            emit ReclaimedStake(fighterId, nrnToReclaim);
        }
    }
```

All winning battles for the new owner will result in a revert. The new owner will have to forsake all `stakeAtRisk` tokens since he cannot claim any tokens.

I'm not sure about the usage of the `amountLost[fighterOwner]` variable. If it's not needed, consider removing it because it will affect the transfers of token owners.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L93-L107

### [L-09] `updateModel()` can be consistently called to increased the `numTrained[tokenId]`

A user can constantly call `updateModel()` with the same modelHash and modelType to increase the `numTrained` variable.

```
    function updateModel(
        uint256 tokenId, 
        string calldata modelHash,
        string calldata modelType
    ) 
        external
    {
        require(msg.sender == ownerOf(tokenId));
        fighters[tokenId].modelHash = modelHash;
        fighters[tokenId].modelType = modelType;
        numTrained[tokenId] += 1;
        totalNumTrained += 1;
    }
```

If `numTrained` affects the elo factor, then there will be a problem as this function is spammable.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L283-L295