# Summary

| Severity                 | Issue                                                                                                       | Instance |
| ------------------------ | ----------------------------------------------------------------------------------------------------------- | -------- |
| [[L-0](#low-0)]          | `iconsTypes` length should also be checked in `FighterFarm::redeemMintPass(...)`                            | 1        |
| [[L-1](#low-1)]          | `customAttributes` are not checked for valid ranges when sent using `FighterFarm::mintFromMergingPool(...)` | 1        |
| [[L-2](#low-2)]          | No check for new Game Item is added in `GameItems::createGameItem(...)`                                     | 1        |
| [[L-3](#low-3)]          | Add checks for duplicate addresses passed as winners in `MergingPool::pickWinner()`                         | 1        |
| [[L-4](#low-4)]          | Add Array parameter length checks in `MergingPool::claimRewards(...)`                                       | 1        |
| [[L-5](#low-5)]          | `Neuron::mint(...)` contract can't mint tokens to the absolute value of `MAX_SUPPLY`                        | 1        |
| [[L-6](#low-6)]          | `globalStakedAmount` is not updated after the end of each round showing incorrect info.                     | 1        |
| [[N-0](#non-critical-0)] | No way to remove a staker in `FighterFarm.sol`                                                              | 1        |
| [[N-1](#non-critical-1)] | Mistake in Natspac of `MergingPool::fighterPoints` mapping.                                                 | 1        |

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
