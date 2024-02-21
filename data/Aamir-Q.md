# Summary

| Severity                 | Issue                                                                                                                                         | Instance |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| [[L-0](#low-0)]          | `iconsTypes` length should also be checked in `FighterFarm::redeemMintPass(...)`                                                              | 1        |
| [[L-1](#low-1)]          | `customAttributes` are not checked for valid ranges when sent using `FighterFarm::mintFromMergingPool(...)`                                   | 1        |
| [[L-2](#low-2)]          | No check for new Game Item is added in `GameItems::createGameItem(...)`                                                                       | 1        |
| [[L-3](#low-3)]          | Add checks for duplicate addresses passed as winners in `MergingPool::pickWinner()`                                                           | 1        |
| [[L-4](#low-4)]          | Add Array parameter length checks in `MergingPool::claimRewards(...)`                                                                         | 1        |
| [[L-5](#low-5)]          | `Neuron::mint(...)` contract can't mint tokens to the absolute value of `MAX_SUPPLY`                                                          | 1        |
| [[L-6](#low-6)]          | `globalStakedAmount` is not updated after the end of each round showing incorrect info.                                                       | 1        |
| [[L-7](#low-7)]          | Holder of the tokenId will not be to claim rewards in `RankedBattle::claimNRN(...)` if the NFT is transferred to another address              | 1        |
| [[L-8](#low-8)]          | `RankedBattle::_getStakingFactor()` works in a biased way for small stakers.                                                                  | 1        |
| [[L-9](#low-9)]          | Only Success case is checked for calls in the contracts that can create issue if the call fails in some cases.                                | 1        |
| [[L-10](#low-10)]        | `MergingPool::claimRewards(...)` can cause DoS if the user won several times but didn't claim the NFT reward immediately                      | 1        |
| [[L-11](#low-11)]        | Various function in `FigtherFarm` allows for putting `modelHash` by the User, but it can harm the Game if something Malicious is put into it. | 1        |
| [[N-0](#non-critical-0)] | No way to remove a staker in `FighterFarm.sol`                                                                                                | 1        |
| [[N-1](#non-critical-1)] | Mistake in Natspac of `MergingPool::fighterPoints` mapping.                                                                                   | 1        |

## Lows

---

### [L-0] `iconsTypes` length should also be checked in `FighterFarm::redeemMintPass(...)` <a id="low-0"></a>

Check for `iconsTypes` length is missed in the FighterFarm::redeemMintPass(...)` contract. This can cause issues later. Add the check fo that as well

```solidity
File: 2024-02-ai-arena/src/FighterFarm.sol

243        require(
244            mintpassIdsToBurn.length == mintPassDnas.length &&
245            mintPassDnas.length == fighterTypes.length &&
246            fighterTypes.length == modelHashes.length &&
248            modelHashes.length == modelTypes.length
249        );
```

GitHub: [[243-248](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L243C1-L248C10)]

---

### [L-1] `customAttributes` are not checked for valid ranges when sent using `FighterFarm::mintFromMergingPool(...)` <a id="low-1"></a>

`FighterFarm::_createFighterBase(...)` allows only setting attributes like `weight` and `element` within specific range. For `weight`, there is a valid range of `65 - 95`. And elements will be selected on the basis of no. of elements set for the generation.

```solidity
File: 2024-02-ai-arena/src/FighterFarm.sol

    function _createFighterBase(
        uint256 dna,
        uint8 fighterType
    )
        private
        view
        returns (uint256, uint256, uint256)
    {
@>        uint256 element = dna % numElements[generation[fighterType]];
@>        uint256 weight = dna % 31 + 65;
@>        uint256 newDna = fighterType == 0 ? dna : uint256(fighterType);
        return (element, weight, newDna);
    }
```

GitHub: ([462-474](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L462C1-L474C6))

But, these ranges will be implemented only when the value inside the `customAttributes` is set to `100`. If any other value is sent then this condition will be used to set the `weight`, `element`, and `dna`:

```solidity
File: 2024-02-ai-arena/src/FighterFarm.sol

502        else {
503            element = customAttributes[0];
504            weight = customAttributes[1];
505            newDna = dna;
506        }
```

GitHub: [[502-506](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L502C1-L506C10)]

This allows anyone to set these values to anything. This can cause issues if the values are set outside the specified range.

#### Recommendations:

Add the neccessary checks for these `customAttributes` in `FighterFarm::_createNewFighter(...)`

```diff
        else {
+           require(customAttributes[0] < numElements[generation[fighterType]]);
+           require(ustomAttributes[1] >= 65 && ustomAttributes[1] <= 95);
            element = customAttributes[0];
            weight = customAttributes[1];
            newDna = dna;
        }
```

---

### [L-2] No check for new Game Item is added in `GameItems::createGameItem(...)` <a id="low-2"></a>

`GameItems::createGameItem(...)` allows admin to create new game items. But there is no check for the different parameters in the functions. If wrong data is sent by mistake it can cause issue. For example, `dailyAllowance` should not be more than the `itemsRemaining`. It is recommended to add the neccessary checks so taht it will not cause any issue later.

---

### [L-3] Add checks for duplicate addresses passed as winners in `MergingPool::pickWinner()` <a id="low-3"></a>

The `MergingPool::pickWinner()` function will used by the Admin to pick the winner. But there is no check in the function whether the duplicate address is passed as a winner in the `winner` paramter. Although the funciton is only admin and he is trusted. But he can still make a mistake and can cause losses to the users by claiming some of the rewards.

```solidity
File: 2024-02-ai-arena/src/MergingPool.sol

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
        roundId += 1;
    }
```

GitHub: [[118-132](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L118C1-L132C6)]

It is recommended to add neccessary checks for that.

---

### [L-4] Add Array parameter length checks in `MergingPool::claimRewards(...)` <a id="low-4"></a>

There is no checks in the `MergingPool::claimRewards(...)` for the length of the array parameters passed. The lengths of the arrays should different array parameters should be matched.

```solidity
File: 2024-02-ai-arena/src/MergingPool.sol

    function claimRewards(
@>        string[] calldata modelURIs,
@>        string[] calldata modelTypes,
@>        uint256[2][] calldata customAttributes
    )
```

GitHub: [[139-143](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139C2-L143C7)]

---

### [L-5] `Neuron::mint(...)` contract can't mint tokens to the absolute value of `MAX_SUPPLY` <a id="low-5"></a>

`Neuron::mint(...)` contract doesn't allow minting of tokens upto absolute `MAX_SUPPLY` because this require statement:

```solidity
File: 2024-02-ai-arena/src/Neuron.sol

    function mint(address to, uint256 amount) public virtual {
@>        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```

GitHub: [[155-159](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L155C1-L159C6)]

As you can see, it uses `<` but should be `<=` so that value upto `MAX_SUPPLY` is included in the amount. It is recommended to add the following check correctly.

```diff
    function mint(address to, uint256 amount) public virtual {
-        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
+        require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```

---

### [L-6] `RankedBattle::globalStakedAmount` is not updated after the end of each round showing incorrect info. <a id="low-6"></a>

When user stake `NRN` in the `RankedBattle` contract, the following two mappings store the staked amount info. `amountStaked` and `globalStakedAmount`. `amountStaked` stores the amount of `NRN` staked token by a specific `tokenId` and `globalStakedAmount` stores the overall amount staked. When user loses/wins the tokens in a battle, some part of the his staked tokens is lost/regained from/to `StakeAtRisk` contract. Also `amountStaked` mapping is updated based on the result. But `globalStakedAmount` in not updated (check in `RankedBattle::updateBattleRecord(...)`). This is because the `globalStakedAmount` is still considered a valid stake as user can reclaim the lost token if he wins a battle. But at the end of each round, any tokens available in `StakeAtRisk` contract are permanently lost. That means `globalStakedAmount` should also be updated on the basis of that. But it is not. That mean `globalStakedAmount` stores incorrect amount.

It is recommended to update the `globalStakedAmount` after the end of each round to match with staked amount. This can be done by making the following changes:

```diff
File: RankedBattle.sol

    function setNewRound() external {
        require(isAdmin[msg.sender]);
        require(totalAccumulatedPoints[roundId] > 0);
+        globalStakedAmount = globalStakedAmount - _stakeAtRiskInstance.totalStakeAtRisk(roundId);
        roundId += 1;
        _stakeAtRiskInstance.setNewRound(roundId);
        rankedNrnDistribution[roundId] = rankedNrnDistribution[roundId - 1];
    }
```

---

### [L-7] Holder of the tokenId will not be to claim rewards in `RankedBattle::claimNRN(...)` if the NFT is transferred to another address <a id="low-7"></a>

If the fighter NFT is transferred to some other address, then the holder of that NFT will not be able to claim the NRN with that new address. Currently the `RankedBattle::claimNRN(...)` only calculates the rewards on the basis of the address of the `msg.sender`, not `tokenId`.

```solidity
File: RankedBattle.sol

    function claimNRN() external {
        require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
        uint256 claimableNRN = 0;
        uint256 nrnDistribution;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
@>                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution
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

GitHub: [[294-311](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L294C1-L311C6)]

This would not be a problem if the user still has access of initial address. But if the user lost the access to the wallet, then he will not be able to claim his rewards for any previous rounds.

It is recommened to calculate the rewards based on the `accumulatedPointsPerFighter` mapping instead. But this would require whole lot of changes in the function. Make sure those are implemented correctly. Doing this can save the funds of user in various situations.

---

### [L-8] `RankedBattle::_getStakingFactor()` works in a biased way for small stakers. <a id="low-8"></a>

The `RankedBattle::_getStakingFactor()` function works little bit biased for small stakers. The staking factor calculated for someone who has staked amount like `3.99 NRN` is same as who has staked `1 wei amount of NRN`. A small staker can take the advantage of this by placing on `1 wei` amount of NRN as a staked and can get benefits of staking equal to someone who has staked `3.99 NRN`. In this way he will not have to worry about losing any big amount and can still avail for the benefits if win.

#### Here is a test for PoC:

NOTE: Include the test below in `RankedBattle.t.sol` File.

```solidity
    function testStakerCanStakeSmallAmountAndGetTheRewardsLikeNormalBiggerStaker() public {
        // setup
        address alice = makeAddr("Alice");
        address bob = makeAddr("bob");

        // giving tokens to alice and bob
        _fundUserWith4kNeuronByTreasury(alice);
        _fundUserWith4kNeuronByTreasury(bob);

        // checking the balance of alice and bob
        assertEq(_neuronContract.balanceOf(alice), 4_000 * 10 ** 18);
        assertEq(_neuronContract.balanceOf(bob), 4_000 * 10 ** 18);

        // minting fighters to alice and bob
        _mintFromMergingPool(alice);
        _mintFromMergingPool(bob);

        // checking if the fighters are minted
        _mintFromMergingPool(alice);
        _mintFromMergingPool(bob);

        // checking if the fighter has been minted
        assertEq(_fighterFarmContract.ownerOf(0), alice);
        assertEq(_fighterFarmContract.ownerOf(1), bob);

        // alice stakes 3.99 NRN in the pool
        // bob stakes only 1 wei amount of NRN in the pool

        vm.prank(alice);
        _rankedBattleContract.stakeNRN(3.99 ether, 0);

        vm.prank(bob);
        _rankedBattleContract.stakeNRN(1 wei, 1);

        // checking if the tokens has been staked
        assertEq(_rankedBattleContract.amountStaked(0), 3.99 ether);
        assertEq(_rankedBattleContract.amountStaked(1), 1 wei);

        // Let's assume alice wins the battle
        vm.startPrank(address(_GAME_SERVER_ADDRESS));
        _rankedBattleContract.updateBattleRecord(0, 0, 0, 1500, true);
        vm.stopPrank();

        // getting the points earned by alice
        uint256 pointsEarnedByAlice = _rankedBattleContract.accumulatedPointsPerAddress(alice,_rankedBattleContract.roundId());

        // Now next round is set
        _rankedBattleContract.setNewRound();

        // Now in this round bob wins
        vm.startPrank(address(_GAME_SERVER_ADDRESS));
        _rankedBattleContract.updateBattleRecord(1, 0, 0, 1500, true);
        vm.stopPrank();

        // getting the points earned by bob
        uint256 pointsEarnedByBob = _rankedBattleContract.accumulatedPointsPerAddress(bob,_rankedBattleContract.roundId());

        // checking if the points earned by bob and alice are equal
        assertEq(pointsEarnedByAlice, pointsEarnedByBob);
    }
```

After running this test, you will see that both Alice and Bob won `1500` points even when bob just stake `1 wei` of NRN while alice staked almost `~4 wei`.

#### Mitigation

Do not allow for staking small amount of tokens. Add a minimum limit for it.

---

### [L-9] Only Success case is checked for calls in the contracts that can create issue if the call fails in some cases. <a id="low-9"></a>

Various calls in the contract only make progress when the transfer call is success, for example:

```solidity
File: RankedBattle.sol

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


            } else {
                /// If the fighter does not have any points for this round, NRNs become at risk of being lost
                bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
@>                if (success) {
                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                    amountStaked[tokenId] -= curStakeAtRisk;
                }
            }
        }
    }

```

GitHub: [[416-500](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L416C1-L500C6)]

As we can see, only when the transfer of token is successfull then the `StakeAtRisk::updateAtRiskRecords()` is called for the staker. But it will not be called if the call is not successfull. And the function will also not revert. This is good thing in some cases, but in some of them it create issue. Like in the above case, if the call fails then the risk Records of the user will not be updated and that would mean user can get away without paying anything from his account when loses. It is always better to add the case when the call fails and the `success` is false. Something like revert if not success wherever needed.

---

### [L-10] `MergingPool::claimRewards(...)` can cause DoS if the user won several times but didn't claim the NFT reward immediately <a id="low-10"></a>

`MergingPool::claimRewards(...)` let user's claim their NFT rewards for the each round ID they won. But it enables them to do so in the same transaction. That mean if the number of claims for the user gets bigger, the function can cause DoS as the function is GAS heavy.

It is recommended to let them claim the rewards for each roundId one by one or specify a max limit that can be claimed in one transaction.

---

### [L-11] Various function in `FigtherFarm` allows for putting `modelHash` by the User, but it can harm the Game if something Malicious is put into it. <a id="low-11"></a>

It is allowed to add the model Hash by the user. But A user can put the hash of something malicious and can harm the game in some way. Add the proper checks to prevent this from happening.

---

## Non Criticals

### [N-0] No way to remove a staker in `FighterFarm.sol` <a id="non-critical-0"></a>

There is only way to add a staker in `FighterFarm.sol` using `addStaker(...)` that can stake the fighters on behalf of the Users. But No way to remove it. This might cause an issue if some contract or address is no longer required to be a staker in the contract. Or if some address is mistakenly added as a staker.

#### Recommendation

Add remove staker function as well:

```diff
+    function removeStaker(address staker) external {
+        require(msg.sender == _ownerAddress);
+        hasStakerRole[staker] = false;
+    }
```

---

### [N-1] Mistake in Natspac for `MergingPool::fighterPoints` mapping. <a id="non-critical-1"></a>

`MergingPool::fighterPoints` is a mapping from `tokenId` to fighter `points` for an NFT. But according to natspac, it is a mapping from `address` to fighter `points` which is incorrect.

```solidity
@>    /// @notice Mapping of address to fighter points.
    mapping(uint256 => uint256) public fighterPoints;
```

GitHub: [[50-52](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L50C1-L52C1)]
