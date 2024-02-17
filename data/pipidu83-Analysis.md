

# Table of Contents
- [Table of Contents](#table-of-contents)
- [Protocol Summary](#protocol-summary)
- [Risk Classification](#risk-classification)
- [Audit Details](#audit-details)
	- [Scope](#scope)
	- [Roles](#roles)
- [Executive Summary](#executive-summary)
	- [Issues found](#issues-found)
- [Findings](#findings)
	- [High](#high)
		- [\[H-1\] `MergingPool::claimRewards` can be DoS-ed, leading to loss of user rewards.](#h-1-mergingpoolclaimrewards-can-be-dos-ed-leading-to-loss-of-user-rewards)
		- [\[H-2\] Staked NRN can stay stuck in the `RankedBattle` contract, leading to loss of user funds.](#h-2-staked-nrn-can-stay-stuck-in-the-rankedbattle-contract-leading-to-loss-of-user-funds)
		- [\[H-3\] The `StakeAtRisk::reclaimNRN` function does not revert on failure, potentially breaking the contract when used in `RankedBattle::_addResultPoints`](#h-3-the-stakeatriskreclaimnrn-function-does-not-revert-on-failure-potentially-breaking-the-contract-when-used-in-rankedbattle_addresultpoints)
		- [\[L-1\] Floating pragma](#l-1-floating-pragma)
		- [\[L-2\] Use of deprecated `_setupRole` function from OpenZeppelin library](#l-2-use-of-deprecated-_setuprole-function-from-openzeppelin-library)
		- [\[NC-1\] Consider using named imports](#nc-1-consider-using-named-imports)
		- [\[NC-2\] Redundant code in `AiArenaHelper::constructor`](#nc-2-redundant-code-in-aiarenahelperconstructor)
		- [\[NC-3\] Avoid using magic numbers](#nc-3-avoid-using-magic-numbers)
		- [\[NC-4\] `require` and `revert` statements should be made more verbose for better readability](#nc-4-require-and-revert-statements-should-be-made-more-verbose-for-better-readability)
		- [\[NC-5\] Missing checks for the zero address](#nc-5-missing-checks-for-the-zero-address)
		- [\[NC-6\] Unused functions marked as `public` should be mared as `external`](#nc-6-unused-functions-marked-as-public-should-be-mared-as-external)

# Protocol Summary

AI arena is a game in which NFT fighters can participate in ranked battles (based on an ELO ranking system) to gain points and get rewards.
The economics of the game are based on the NRN Neuron token. Players can stake their NRN to earn more NRN rewards by playing in ranked battles.

Each ranked battle is an opportunity for players to get points. If a player has a positive number of points at the end of a round, he can convert those points into NRN rewards.
If does not have points and loses a ranked battle, this will result in part of their staked NRN to be placed "at risk" meaning they will be lost if they player does not make back the lost points by winning battles before the end of the round.

Engaging in ranked battles necessitates voltage. Voltage for each player is replenished every 24 hours or can be replenished before that via the purchase of batteries from the Game Items contract.

They are also able to stake their fighter in the Merging Pool to get a chance to mint a new NFT. Their probability to win a new NFT via the Merging Pool is determined by the amount of points they decided to allocate to the merging pool prior to winning a ranked battle.

# Risk Classification

We use the below matrix based on Impact and Likelihood to determine the severity of each issue detailed in this analysis report.

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H      | M   |
| Likelihood | Medium | H      | M      | M   |
|            | Low    | M      | M      | L   |

# Audit Details

## Scope

There is a total of 1278 SLOC in scope throughout the below 9 files.

```
./src/
__AiArenaHelper.sol
__FighterFarm.sol
__FighterOps.sol
__GameItems.sol
__MergingPool.sol
__Neuron.sol
__RankedBattle.sol
__StakeAtRisk.sol
__VoltageManager.sol
```

## Roles

Owner - Deployer of the protocol. Owner has a range of rights including but not limited to setting minter, spender and staker roles defined below. Additionally he can transfer ownership of the contract to another address, add attribute divisors for fighter attributes, add/delete attribute probabilities, increment fighter generations, grant staker role, instantiate AIArenaHelper, MintPass, Neuron and Merging Pool contracts in the Fighter Farm contract.

Admin Role - The admins have a range of rights on the protocol, including updating game items and updating transferrability of these items.

Minter Role - has the ability to mint Neuron tokens to users.

Spender Role - has the ability to spend Neuron tokens and to grant this ability to other users.

Staker Role - has the ability to add stakers to the Neuron contract.

# Executive Summary

While on a high level we find the overall organisation of the game to be satisfactory and the breakdown by the different smart contracts to be coherent, we believe some more attention should be granted to the design and logic of functions within these smart contracts.

More particularly, the `transfer` and `transferFrom` function used on the ERC20 `Neuron` contract are highly sensitive and inproper checks on their return values, associated with a misues of the Checks-Effects-Interactions pattern could unnecessarily put these smart contracts at risk of being attacked/broken by malicious users and/or by silent failures in normal use of the contracts.

We notice throughout the smart contracts in scope that no action is taken when these functions return a value of `false` (we reported one of these occurences as a HIGH vulnerability as it could directly impact the economics of the game). We hence recommend always checking for the return values of these functions and revert any action taken within the same function by reverting the whole transaction instead of completing it silently. An alternative to this structure would be to use the [`SafeER20`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol) library from OpenZeppelin.

More attention could also be paid to the "finishing phase" to produce a production-ready code. This includes avoiding the use of magic numbers that should be defined as constants for better readability, the use of floating solidity versions that could make the protocol vulnerable to outdated versions of the compiler for example (using a fixed pragma would also add confidence that the protocol has been throughly tested with a given version of solidity), the use of unnamed imports and checks for the zero address. It also be useful to add more verbosity to errors and reverts to simplifiy the process of debugging and understanding of the protocol by external users.

On the centralization risks, we notice that a large range of abilities is granted to the owner role throughout the contracts of the protocol, putting it at risk of being manipulated if this role gets compromised / if owner can no more be trusted. For added security, the owner could consider revoking ownership. This is conditional on the fact that other roles are stricly limited to their perimeters and that the protocol can function without a single point of centralized ownership.

Lastly, we believe it would make sense to include the `Verification` library in scope for this audit as it could contain vulnerabilities induced in particular by the use of the `ecrecover` function, and is being used in the `claimFighters` function of the `FighterFarm` contract which is of key importance in the protocol.




## Issues found

| Severity | Number of issues found |
| -------- | ---------------------- |
| High     | 3                      |
| Medium   | 0                      |
| Low      | 2                      |
| NC       | 6                      |
| Total    | 11                     |

# Findings

## High

### [H-1] `MergingPool::claimRewards` can be DoS-ed, leading to loss of user rewards.

**Impact**

The `MergingPool::claimRewards` function can be used by winners to claim multiple rounds of rewards. However, looping through each winning address for each round is inefficient and will lead to increasing gas costs. If a winner does not claim rewards for a large number of rounds, the function will become too gas intensive to be used and user will not be able to claim her rewards.
This situation is likely to happen when number of rounds and/or number of players/winners grow (number of winners for each round can be increased by owner via the `MergingPool::updateWinnersPerPeriod` function), and has severe consequences as it means users may be denied their rewards. We then classify this vulnerability as HIGH.

As a side note, `MergingPool::getUnclaimedRewards` will be DoS-ed in the same way, although it is less critical in that case as not leading to loss of rewards for winning users.

**Proof of Concept**

We prove in the Proof of Code below that gas cost will increase with the number of unclaimed rounds for a winning user. For the sake of simplicity, we consider the below scenario:

1. `_ownerAddress` is picked as a winner a first time.
2. `_ownerAddress` calls `MergingPool::claimRewards` a first time, leading to `gasCostA`
3. `_ownerAddress` is picked as a winner for two subsequent rounds.
4. `_ownerAddress` then has two rounds of unclaimed rewards, which they claim using the `MergingPool::claimRewards` function a second time, associated to `gasCostB`
5. We then verifiy that `gasCostB > gasCostA` meaning it is more gas intensive to claim two rounds of rewards than one.

By generalizing the above, claiming `n` rounds of rewards if going to be more gas intensive than claiming `n-1` rounds of rewards, meaning gas costs increase with the number of unclaimed rounds and will end up making `MergingPool::claimRewards` too costly to use.
We took the example where user claiming rewards has been picked as winner for all rounds but the same works in case they are not.

<details>

<summary>PoC</summary>

Paste the below code in `MergingPool.t.sol`

```javascript
    function testClaimRewardsForWinnersOfMultipleRoundIdsCanBeDoSedSimplified() public {

        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_DELEGATED_ADDRESS);

        uint256[] memory _winners = new uint256[](2);
        _winners[0] = 0;
        _winners[1] = 1;
        // winners of roundId 0 are picked
        _mergingPoolContract.pickWinner(_winners);
        assertEq(_mergingPoolContract.isSelectionComplete(0), true);

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

        uint256 gasStart = gasleft();
        _mergingPoolContract.claimRewards(_modelURIs, _modelTypes, _customAttributes);
        uint256 gasCostA = gasStart - gasleft();

        _mergingPoolContract.pickWinner(_winners);
        _mergingPoolContract.pickWinner(_winners);

        uint256 gasStart2 = gasleft();
        _mergingPoolContract.claimRewards(_modelURIs, _modelTypes, _customAttributes);
        uint256 gasCostB = gasStart2 - gasleft();

        assert(gasCostB > gasCostA);
    }
```


</details>

**Tools Used**

Manual Review / Foundry

**Recommended Mitigation Steps**

To mitigate this issue, we could define a mapping that keeps track of unclaimed rewards for each user. With this mechanism, we would keep track of the total number of unclaimed wins for each user, which would go up each time this user wins, and down by the number of claimed rewards each time this user claims them.

This way, we could make the `MergingPool::claimRewards` function much more gas efficient and decreased the likelihood of DoS.

### [H-2] Staked NRN can stay stuck in the `RankedBattle` contract, leading to loss of user funds.

**Impact**

In the `RankedBattle::unstakeNRN` function, no action is taken when `_neuronInstance.transfer(msg.sender, amount)` returns `false`. 
This means that the code below will get executed even in the event of a failed transfer of NRN back to the user.

```javascript
        amountStaked[tokenId] -= amount;
        globalStakedAmount -= amount;
        stakingFactor[tokenId] = _getStakingFactor(
            tokenId, 
            _stakeAtRiskInstance.getStakeAtRisk(tokenId)
        );
        _calculatedStakingFactor[tokenId][roundId] = true;
        hasUnstaked[tokenId][roundId] = true;
```

The staked NRN will then effectively be stuck in the `RankedBattle` contract, making it impossible for the user to get them back and to benefit from having their NRN staked.


**Proof of Concept**

Let's say Bob has 1000 NRN staked in the `RankedBattle` contract for `tokenId` 1. 

1. Bob calls `unstakeNRN(1000, 1)` to get them back.
2. `_neuronInstance.transfer(msg.sender, amount)` returns false and trasnfer of NRN back to Bob fails.
3. `amountStaked[tokenId]` is now 0, `hasUnstaked[tokenId][roundId]` is `true`, meaning Bob cannot call `stakeNRN` because of the `require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");` condition, and cannot call `unstakeNRN` another time to get his NRN back as the code below will ensure he cannot claim more than `0` NRN.

```javascript
        if (amount > amountStaked[tokenId]) {
            amount = amountStaked[tokenId];
        }
```

This means his NRN are effectively stuck in the contract.

**Tools Used**

Manual Review / Visual Studio

**Recommended Mitigation Steps**

The transaction should revert in case of transfer failure. This will prevent `amountStaked[tokenId]`, `globalStakedAmount` and `hasUnstaked[tokenId][roundId]` from being incorrectly set.

```diff
        bool success = _neuronInstance.transfer(msg.sender, amount);
        if (success) {
            if (amountStaked[tokenId] == 0) {
                _fighterFarmInstance.updateFighterStaking(tokenId, false);
            }
            emit Unstaked(msg.sender, amount);
        }
+		else {
+			revert();
+		}
```


### [H-3] The `StakeAtRisk::reclaimNRN` function does not revert on failure, potentially breaking the contract when used in `RankedBattle::_addResultPoints` 

**Impact**

In the `RankedBattle::_addResultPoints`, if user won the match (i.e. if `battleResult == 0`) then we execute the below code

```javascript
if (curStakeAtRisk > 0) {
                _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
                amountStaked[tokenId] += curStakeAtRisk;
    }
```

However if we take a look at the `StakeAtRisk::reclaimNRN` we see that the transfer of NRN back the the `_rankedBattleAddress` could silently fail as we do not revert on failure.

```javascript
        bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
        if (success) {
            stakeAtRisk[roundId][fighterId] -= nrnToReclaim;
            totalStakeAtRisk[roundId] -= nrnToReclaim;
            amountLost[fighterOwner] -= nrnToReclaim;
            emit ReclaimedStake(fighterId, nrnToReclaim);
        }
```

This means that we will still execute `amountStaked[tokenId] += curStakeAtRisk` even though the `curStakeAtRisk` amount was never transferred back to the `RankedBattle` contract.

This leads to erronated `amountStaked[tokenId]` which is used in many key parts of the contract, effectively breaking the economics of the protocol. This is then a HIGH vulnerability


**Proof of Concept**

1. `amountStaked[1]` > 0, Bob wins against Alice for `tokenId == 1`
2. Game Server calls `updateBattleRecord` which executes the line below

```javascript
        _addResultPoints(1, 1, eloFactor, mergingPortion < 100, Bob);
```

3. Assume `curStakeAtRisk > 0` we execute inside the `_addResultPoints` function:

```javascript
        _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, 1, Bob);
        amountStaked[1] += curStakeAtRisk;
```

4. Inside the `reclaimNRN` if the line below fails, we end up with 

```javascript
amountStaked[1] += curStakeAtRisk;
```

still being executed meaning `amountStaked[1]` will be wrong, impacting all subsequent actions for this tokenId.

**Tools Used**

Manual Review / Visual Studio

**Recommended Mitigation Steps**

The mitigation step is straightforward and we could simply revert on failure in the `StakeAtRisk::reclaimNRN` function like below

```diff
        bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
        if (success) {
            stakeAtRisk[roundId][fighterId] -= nrnToReclaim;
            totalStakeAtRisk[roundId] -= nrnToReclaim;
            amountLost[fighterOwner] -= nrnToReclaim;
            emit ReclaimedStake(fighterId, nrnToReclaim);
        }
+		else {
+			revert();
+		}
```

### [L-1] Floating pragma

Consider using specific versions of Solidity instead of floating pragma, this will ensure that contracts do not get deployed with outdated compiler versions for examples.

See the SWC Registry for more info on this [issue](https://swcregistry.io/docs/SWC-103/).


- Found in src/AiArenaHelper.sol [Line: 3](src/AiArenaHelper.sol#L3)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```

- Found in src/FighterFarm.sol [Line: 2](src/FighterFarm.sol#L2)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```

- Found in src/FighterOps.sol [Line: 2](src/FighterOps.sol#L2)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```


- Found in src/GameItems.sol [Line: 2](src/GameItems.sol#L2)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```

- Found in src/MergingPool.sol [Line: 2](src/MergingPool.sol#L2)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```

- Found in src/Neuron.sol [Line: 2](src/Neuron.sol#L2)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```

- Found in src/RankedBattle.sol [Line: 2](src/RankedBattle.sol#L2)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```

- Found in src/StakeAtRisk.sol [Line: 2](src/StakeAtRisk.sol#L2)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```


- Found in src/VoltageManager.sol [Line: 2](src/VoltageManager.sol#L2)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```

### [L-2] Use of deprecated `_setupRole` function from OpenZeppelin library

Consider replacing this function with newer versions introduced by OpenZeppelin.

- Found in src/Neuron.sol [Line: 97](src/Neuron.sol#L97)

	```solidity
	        _setupRole(MINTER_ROLE, newMinterAddress);
	```

- Found in src/Neuron.sol [Line: 105](src/Neuron.sol#L105)

	```solidity
	        _setupRole(STAKER_ROLE, newStakerAddress);
	```

- Found in src/Neuron.sol [Line: 113](src/Neuron.sol#L113)

	```solidity
	        _setupRole(SPENDER_ROLE, newSpenderAddress);
	```


### [NC-1] Consider using named imports

This will add readability for developers and auditors, and make it easier to spot inefficiencies.

- Found in src/FighterFarm.sol [Line: 9](src/FighterFarm.sol#L9)

    ```solidity
    import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
    ```

- Found in src/FighterFarm.sol [Line: 10](src/FighterFarm.sol#L10)

    ```solidity
    import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
    ```

- Found in src/GameItems.sol [Line: 5](src/GameItems.sol#L5)

    ```solidity
    import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
    ```

- Found in src/Neuron.sol [Line: 6](src/Neuron.sol#L6)

    ```solidity
    import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
    ```

- Found in src/Neuron.sol [Line: 7](src/Neuron.sol#L7)

    ```solidity
    import "@openzeppelin/contracts/access/AccessControl.sol";
    ```

### [NC-2] Redundant code in `AiArenaHelper::constructor`

In `AiArenaHelper::constructor`, the below line is useless and is redundant with the lines below it. It is a waste of gas and is bad for readability of the constructor. Consider removing it.

```diff
-        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
```

### [NC-3] Avoid using magic numbers

This will increase readability and could prevent mistakes in the use of constant values.

- Found in src/AiArenaHelper.sol [Line: 109](src/AiArenaHelper.sol#L109)

	```solidity
	            return FighterOps.FighterPhysicalAttributes(99, 99, 99, 99, 99, 99);
	```

- Found in src/AiArenaHelper.sol [Line: 116](src/AiArenaHelper.sol#L116)

	```solidity
	                  i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
	```

- Found in src/AiArenaHelper.sol [Line: 117](src/AiArenaHelper.sol#L117)

	```solidity
	                  i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
	```

- Found in src/AiArenaHelper.sol [Line: 118](src/AiArenaHelper.sol#L118)

	```solidity
	                  i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
	```

- Found in src/AiArenaHelper.sol [Line: 120](src/AiArenaHelper.sol#L120)

	```solidity
	                    finalAttributeProbabilityIndexes[i] = 50;
	```

- Found in src/AiArenaHelper.sol [Line: 124](src/AiArenaHelper.sol#L124)

	```solidity
	                    uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;
	```

- Found in src/AiArenaHelper.sol [Line: 131](src/AiArenaHelper.sol#L131)

	```solidity
	                finalAttributeProbabilityIndexes[1],
	```

- Found in src/AiArenaHelper.sol [Line: 132](src/AiArenaHelper.sol#L132)

	```solidity
	                finalAttributeProbabilityIndexes[2],
	```

- Found in src/AiArenaHelper.sol [Line: 133](src/AiArenaHelper.sol#L133)

	```solidity
	                finalAttributeProbabilityIndexes[3],
	```

- Found in src/AiArenaHelper.sol [Line: 134](src/AiArenaHelper.sol#L134)

	```solidity
	                finalAttributeProbabilityIndexes[4],
	```

- Found in src/AiArenaHelper.sol [Line: 135](src/AiArenaHelper.sol#L135)

	```solidity
	                finalAttributeProbabilityIndexes[5]
	```

- Found in src/AiArenaHelper.sol [Line: 151](src/AiArenaHelper.sol#L151)

	```solidity
	        require(probabilities.length == 6, "Invalid number of attribute arrays");
	```

- Found in src/AiArenaHelper.sol [Line: 199](src/AiArenaHelper.sol#L199)

	```solidity
	                attributeProbabilityIndex = i + 1;
	```

- Found in src/FighterFarm.sol [Line: 110](src/FighterFarm.sol#L110)

	```solidity
	        numElements[0] = 3;
	```

- Found in src/FighterFarm.sol [Line: 131](src/FighterFarm.sol#L131)

	```solidity
	        generation[fighterType] += 1;
	```

- Found in src/FighterFarm.sol [Line: 132](src/FighterFarm.sol#L132)

	```solidity
	        maxRerollsAllowed[fighterType] += 1;
	```

- Found in src/FighterFarm.sol [Line: 192](src/FighterFarm.sol#L192)

	```solidity
	        uint8[2] calldata numToMint,
	```

- Found in src/FighterFarm.sol [Line: 202](src/FighterFarm.sol#L202)

	```solidity
	            numToMint[1],
	```

- Found in src/FighterFarm.sol [Line: 204](src/FighterFarm.sol#L204)

	```solidity
	            nftsClaimed[msg.sender][1]
	```

- Found in src/FighterFarm.sol [Line: 207](src/FighterFarm.sol#L207)

	```solidity
	        uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
	```

- Found in src/FighterFarm.sol [Line: 210](src/FighterFarm.sol#L210)

	```solidity
	        nftsClaimed[msg.sender][1] += numToMint[1];
	```

- Found in src/FighterFarm.sol [Line: 217](src/FighterFarm.sol#L217)

	```solidity
	                i < numToMint[0] ? 0 : 1,
	```

- Found in src/FighterFarm.sol [Line: 219](src/FighterFarm.sol#L219)

	```solidity
	                [uint256(100), uint256(100)]
	```

- Found in src/FighterFarm.sol [Line: 259](src/FighterFarm.sol#L259)

	```solidity
	                [uint256(100), uint256(100)]
	```

- Found in src/FighterFarm.sol [Line: 293](src/FighterFarm.sol#L293)

	```solidity
	        numTrained[tokenId] += 1;
	```

- Found in src/FighterFarm.sol [Line: 294](src/FighterFarm.sol#L294)

	```solidity
	        totalNumTrained += 1;
	```

- Found in src/FighterFarm.sol [Line: 317](src/FighterFarm.sol#L317)

	```solidity
	        uint256[2] calldata customAttributes
	```

- Found in src/FighterFarm.sol [Line: 378](src/FighterFarm.sol#L378)

	```solidity
	            numRerolls[tokenId] += 1;
	```

- Found in src/FighterFarm.sol [Line: 428](src/FighterFarm.sol#L428)

	```solidity
	            uint256[6] memory,
	```

- Found in src/FighterFarm.sol [Line: 472](src/FighterFarm.sol#L472)

	```solidity
	        uint256 weight = dna % 31 + 65;
	```

- Found in src/FighterFarm.sol [Line: 492](src/FighterFarm.sol#L492)

	```solidity
	        uint256[2] memory customAttributes
	```

- Found in src/FighterFarm.sol [Line: 500](src/FighterFarm.sol#L500)

	```solidity
	        if (customAttributes[0] == 100) {
	```

- Found in src/FighterFarm.sol [Line: 505](src/FighterFarm.sol#L505)

	```solidity
	            weight = customAttributes[1];
	```

- Found in src/FighterFarm.sol [Line: 510](src/FighterFarm.sol#L510)

	```solidity
	        bool dendroidBool = fighterType == 1;
	```

- Found in src/FighterOps.sol [Line: 72](src/FighterOps.sol#L72)

	```solidity
	    function getFighterAttributes(Fighter storage self) public view returns (uint256[6] memory) {
	```

- Found in src/FighterOps.sol [Line: 92](src/FighterOps.sol#L92)

	```solidity
	            uint256[6] memory,
	```

- Found in src/GameItems.sol [Line: 238](src/GameItems.sol#L238)

	```solidity
	        _itemCount += 1;
	```

- Found in src/GameItems.sol [Line: 318](src/GameItems.sol#L318)

	```solidity
	        dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
	```

- Found in src/MergingPool.sol [Line: 131](src/MergingPool.sol#L131)

	```solidity
	        roundId += 1;
	```

- Found in src/MergingPool.sol [Line: 142](src/MergingPool.sol#L142)

	```solidity
	        uint256[2][] calldata customAttributes
	```

- Found in src/MergingPool.sol [Line: 154](src/MergingPool.sol#L154)

	```solidity
	            numRoundsClaimed[msg.sender] += 1;
	```

- Found in src/MergingPool.sol [Line: 165](src/MergingPool.sol#L165)

	```solidity
	                    claimIndex += 1;
	```

- Found in src/MergingPool.sol [Line: 186](src/MergingPool.sol#L186)

	```solidity
	                    numRewards += 1;
	```

- Found in src/MergingPool.sol [Line: 212](src/MergingPool.sol#L212)

	```solidity
	        uint256[] memory points = new uint256[](1);
	```

- Found in src/RankedBattle.sol [Line: 157](src/RankedBattle.sol#L157)

	```solidity
	        rankedNrnDistribution[0] = 5000 * 10**18;
	```

- Found in src/RankedBattle.sol [Line: 220](src/RankedBattle.sol#L220)

	```solidity
	        rankedNrnDistribution[roundId] = newDistribution * 10**18;
	```

- Found in src/RankedBattle.sol [Line: 236](src/RankedBattle.sol#L236)

	```solidity
	        roundId += 1;
	```

- Found in src/RankedBattle.sol [Line: 238](src/RankedBattle.sol#L238)

	```solidity
	        rankedNrnDistribution[roundId] = rankedNrnDistribution[roundId - 1];
	```

- Found in src/RankedBattle.sol [Line: 305](src/RankedBattle.sol#L305)

	```solidity
	            numRoundsClaimed[msg.sender] += 1;
	```

- Found in src/RankedBattle.sol [Line: 334](src/RankedBattle.sol#L334)

	```solidity
	        require(mergingPortion <= 100);
	```

- Found in src/RankedBattle.sol [Line: 350](src/RankedBattle.sol#L350)

	```solidity
	        totalBattles += 1;
	```

- Found in src/RankedBattle.sol [Line: 442](src/RankedBattle.sol#L442)

	```solidity
	        curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
	```

- Found in src/RankedBattle.sol [Line: 453](src/RankedBattle.sol#L453)

	```solidity
	            uint256 mergingPoints = (points * mergingPortion) / 100;
	```

- Found in src/RankedBattle.sol [Line: 479](src/RankedBattle.sol#L479)

	```solidity
	        } else if (battleResult == 2) {
	```

- Found in src/RankedBattle.sol [Line: 516](src/RankedBattle.sol#L516)

	```solidity
	            fighterBattleRecord[tokenId].wins += 1;
	```

- Found in src/RankedBattle.sol [Line: 517](src/RankedBattle.sol#L517)

	```solidity
	        } else if (battleResult == 1) {
	```

- Found in src/RankedBattle.sol [Line: 518](src/RankedBattle.sol#L518)

	```solidity
	            fighterBattleRecord[tokenId].ties += 1;
	```

- Found in src/RankedBattle.sol [Line: 519](src/RankedBattle.sol#L519)

	```solidity
	        } else if (battleResult == 2) {
	```

- Found in src/RankedBattle.sol [Line: 520](src/RankedBattle.sol#L520)

	```solidity
	            fighterBattleRecord[tokenId].loses += 1;
	```

- Found in src/RankedBattle.sol [Line: 537](src/RankedBattle.sol#L537)

	```solidity
	          (amountStaked[tokenId] + stakeAtRisk) / 10**18
	```

- Found in src/RankedBattle.sol [Line: 540](src/RankedBattle.sol#L540)

	```solidity
	        stakingFactor_ = 1;
	```

- Found in src/VoltageManager.sol [Line: 95](src/VoltageManager.sol#L95)

	```solidity
	        require(ownerVoltage[msg.sender] < 100);
	```

- Found in src/VoltageManager.sol [Line: 97](src/VoltageManager.sol#L97)

	```solidity
	        _gameItemsContractInstance.burn(msg.sender, 0, 1);
	```

- Found in src/VoltageManager.sol [Line: 98](src/VoltageManager.sol#L98)

	```solidity
	        ownerVoltage[msg.sender] = 100;
	```

- Found in src/VoltageManager.sol [Line: 120](src/VoltageManager.sol#L120)

	```solidity
	        ownerVoltage[owner] = 100;
	```

- Found in src/VoltageManager.sol [Line: 121](src/VoltageManager.sol#L121)

	```solidity
	        ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days);
	```


### [NC-4] `require` and `revert` statements should be made more verbose for better readability

- Found in src/AiArenaHelper.sol [Line: 77](src/AiArenaHelper.sol#L77)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/AiArenaHelper.sol [Line: 84](src/AiArenaHelper.sol#L84)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/AiArenaHelper.sol [Line: 85](src/AiArenaHelper.sol#L85)

	```solidity
	        require(attributeDivisors.length == attributes.length);
	```

- Found in src/AiArenaHelper.sol [Line: 149](src/AiArenaHelper.sol#L149)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/AiArenaHelper.sol [Line: 163](src/AiArenaHelper.sol#L163)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/FighterFarm.sol [Line: 121](src/FighterFarm.sol#L121)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/FighterFarm.sol [Line: 130](src/FighterFarm.sol#L130)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/FighterFarm.sol [Line: 140](src/FighterFarm.sol#L140)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/FighterFarm.sol [Line: 148](src/FighterFarm.sol#L148)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/FighterFarm.sol [Line: 156](src/FighterFarm.sol#L156)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/FighterFarm.sol [Line: 164](src/FighterFarm.sol#L164)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/FighterFarm.sol [Line: 172](src/FighterFarm.sol#L172)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/FighterFarm.sol [Line: 181](src/FighterFarm.sol#L181)

	```solidity
	        require(msg.sender == _delegatedAddress);
	```

- Found in src/FighterFarm.sol [Line: 206](src/FighterFarm.sol#L206)

	```solidity
	        require(Verification.verify(msgHash, signature, _delegatedAddress));
	```

- Found in src/FighterFarm.sol [Line: 208](src/FighterFarm.sol#L208)

	```solidity
	        require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
	```

- Found in src/FighterFarm.sol [Line: 243](src/FighterFarm.sol#L243)

	```solidity
	        require(
	```

- Found in src/FighterFarm.sol [Line: 250](src/FighterFarm.sol#L250)

	```solidity
	            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
	```

- Found in src/FighterFarm.sol [Line: 269](src/FighterFarm.sol#L269)

	```solidity
	        require(hasStakerRole[msg.sender]);
	```

- Found in src/FighterFarm.sol [Line: 290](src/FighterFarm.sol#L290)

	```solidity
	        require(msg.sender == ownerOf(tokenId));
	```

- Found in src/FighterFarm.sol [Line: 321](src/FighterFarm.sol#L321)

	```solidity
	        require(msg.sender == _mergingPoolAddress);
	```

- Found in src/FighterFarm.sol [Line: 346](src/FighterFarm.sol#L346)

	```solidity
	        require(_ableToTransfer(tokenId, to));
	```

- Found in src/FighterFarm.sol [Line: 363](src/FighterFarm.sol#L363)

	```solidity
	        require(_ableToTransfer(tokenId, to));
	```

- Found in src/FighterFarm.sol [Line: 371](src/FighterFarm.sol#L371)

	```solidity
	        require(msg.sender == ownerOf(tokenId));
	```

- Found in src/FighterFarm.sol [Line: 372](src/FighterFarm.sol#L372)

	```solidity
	        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
	```

- Found in src/FighterFarm.sol [Line: 496](src/FighterFarm.sol#L496)

	```solidity
	        require(balanceOf(to) < MAX_FIGHTERS_ALLOWED);
	```

- Found in src/GameItems.sol [Line: 109](src/GameItems.sol#L109)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/GameItems.sol [Line: 118](src/GameItems.sol#L118)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/GameItems.sol [Line: 127](src/GameItems.sol#L127)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/GameItems.sol [Line: 140](src/GameItems.sol#L140)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/GameItems.sol [Line: 148](src/GameItems.sol#L148)

	```solidity
	        require(tokenId < _itemCount);
	```

- Found in src/GameItems.sol [Line: 151](src/GameItems.sol#L151)

	```solidity
	        require(
	```

- Found in src/GameItems.sol [Line: 160](src/GameItems.sol#L160)

	```solidity
	        require(
	```

- Found in src/GameItems.sol [Line: 190](src/GameItems.sol#L190)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/GameItems.sol [Line: 199](src/GameItems.sol#L199)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/GameItems.sol [Line: 223](src/GameItems.sol#L223)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/GameItems.sol [Line: 247](src/GameItems.sol#L247)

	```solidity
	        require(allowedBurningAddresses[msg.sender]);
	```

- Found in src/GameItems.sol [Line: 305](src/GameItems.sol#L305)

	```solidity
	        require(allGameItemAttributes[tokenId].transferable);
	```

- Found in src/MergingPool.sol [Line: 90](src/MergingPool.sol#L90)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/MergingPool.sol [Line: 99](src/MergingPool.sol#L99)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/MergingPool.sol [Line: 107](src/MergingPool.sol#L107)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/MergingPool.sol [Line: 119](src/MergingPool.sol#L119)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/Neuron.sol [Line: 88](src/Neuron.sol#L88)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/Neuron.sol [Line: 96](src/Neuron.sol#L96)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/Neuron.sol [Line: 104](src/Neuron.sol#L104)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/Neuron.sol [Line: 112](src/Neuron.sol#L112)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/Neuron.sol [Line: 121](src/Neuron.sol#L121)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/Neuron.sol [Line: 130](src/Neuron.sol#L130)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/Neuron.sol [Line: 131](src/Neuron.sol#L131)

	```solidity
	        require(recipients.length == amounts.length);
	```

- Found in src/RankedBattle.sol [Line: 168](src/RankedBattle.sol#L168)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/RankedBattle.sol [Line: 177](src/RankedBattle.sol#L177)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/RankedBattle.sol [Line: 185](src/RankedBattle.sol#L185)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/RankedBattle.sol [Line: 193](src/RankedBattle.sol#L193)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/RankedBattle.sol [Line: 202](src/RankedBattle.sol#L202)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/RankedBattle.sol [Line: 210](src/RankedBattle.sol#L210)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/RankedBattle.sol [Line: 219](src/RankedBattle.sol#L219)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/RankedBattle.sol [Line: 227](src/RankedBattle.sol#L227)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/RankedBattle.sol [Line: 234](src/RankedBattle.sol#L234)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/RankedBattle.sol [Line: 235](src/RankedBattle.sol#L235)

	```solidity
	        require(totalAccumulatedPoints[roundId] > 0);
	```

- Found in src/RankedBattle.sol [Line: 333](src/RankedBattle.sol#L333)

	```solidity
	        require(msg.sender == _gameServerAddress);
	```

- Found in src/RankedBattle.sol [Line: 334](src/RankedBattle.sol#L334)

	```solidity
	        require(mergingPortion <= 100);
	```

- Found in src/RankedBattle.sol [Line: 336](src/RankedBattle.sol#L336)

	```solidity
	        require(
	```

- Found in src/VoltageManager.sol [Line: 66](src/VoltageManager.sol#L66)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/VoltageManager.sol [Line: 75](src/VoltageManager.sol#L75)

	```solidity
	        require(msg.sender == _ownerAddress);
	```

- Found in src/VoltageManager.sol [Line: 84](src/VoltageManager.sol#L84)

	```solidity
	        require(isAdmin[msg.sender]);
	```

- Found in src/VoltageManager.sol [Line: 95](src/VoltageManager.sol#L95)

	```solidity
	        require(ownerVoltage[msg.sender] < 100);
	```

- Found in src/VoltageManager.sol [Line: 96](src/VoltageManager.sol#L96)

	```solidity
	        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
	```

- Found in src/VoltageManager.sol [Line: 108](src/VoltageManager.sol#L108)

	```solidity
	        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
	```

### [NC-5] Missing checks for the zero address

- Found in src/AiArenaHelper.sol [Line: 78](src/AiArenaHelper.sol#L78)

	```solidity
	        _ownerAddress = newOwnerAddress;
	```

- Found in src/FighterFarm.sol [Line: 107](src/FighterFarm.sol#L107)

	```solidity
	        _ownerAddress = ownerAddress;
	```

- Found in src/FighterFarm.sol [Line: 108](src/FighterFarm.sol#L108)

	```solidity
	        _delegatedAddress = delegatedAddress;
	```

- Found in src/FighterFarm.sol [Line: 109](src/FighterFarm.sol#L109)

	```solidity
	        treasuryAddress = treasuryAddress_;
	```

- Found in src/FighterFarm.sol [Line: 122](src/FighterFarm.sol#L122)

	```solidity
	        _ownerAddress = newOwnerAddress;
	```

- Found in src/FighterFarm.sol [Line: 173](src/FighterFarm.sol#L173)

	```solidity
	        _mergingPoolAddress = mergingPoolAddress;
	```

- Found in src/GameItems.sol [Line: 96](src/GameItems.sol#L96)

	```solidity
	        _ownerAddress = ownerAddress;
	```

- Found in src/GameItems.sol [Line: 97](src/GameItems.sol#L97)

	```solidity
	        treasuryAddress = treasuryAddress_;
	```

- Found in src/GameItems.sol [Line: 110](src/GameItems.sol#L110)

	```solidity
	        _ownerAddress = newOwnerAddress;
	```

- Found in src/MergingPool.sol [Line: 76](src/MergingPool.sol#L76)

	```solidity
	        _ownerAddress = ownerAddress;
	```

- Found in src/MergingPool.sol [Line: 77](src/MergingPool.sol#L77)

	```solidity
	        _rankedBattleAddress = rankedBattleAddress;
	```

- Found in src/MergingPool.sol [Line: 91](src/MergingPool.sol#L91)

	```solidity
	        _ownerAddress = newOwnerAddress;
	```

- Found in src/Neuron.sol [Line: 73](src/Neuron.sol#L73)

	```solidity
	        _ownerAddress = ownerAddress;
	```

- Found in src/Neuron.sol [Line: 74](src/Neuron.sol#L74)

	```solidity
	        treasuryAddress = treasuryAddress_;
	```

- Found in src/Neuron.sol [Line: 89](src/Neuron.sol#L89)

	```solidity
	        _ownerAddress = newOwnerAddress;
	```

- Found in src/RankedBattle.sol [Line: 152](src/RankedBattle.sol#L152)

	```solidity
	        _ownerAddress = ownerAddress;
	```

- Found in src/RankedBattle.sol [Line: 153](src/RankedBattle.sol#L153)

	```solidity
	        _gameServerAddress = gameServerAddress;
	```

- Found in src/RankedBattle.sol [Line: 169](src/RankedBattle.sol#L169)

	```solidity
	        _ownerAddress = newOwnerAddress;
	```

- Found in src/RankedBattle.sol [Line: 186](src/RankedBattle.sol#L186)

	```solidity
	        _gameServerAddress = gameServerAddress;
	```

- Found in src/RankedBattle.sol [Line: 194](src/RankedBattle.sol#L194)

	```solidity
	        _stakeAtRiskAddress = stakeAtRiskAddress;
	```

- Found in src/StakeAtRisk.sol [Line: 65](src/StakeAtRisk.sol#L65)

	```solidity
	        treasuryAddress = treasuryAddress_;
	```

- Found in src/StakeAtRisk.sol [Line: 66](src/StakeAtRisk.sol#L66)

	```solidity
	        _rankedBattleAddress = rankedBattleAddress;   
	```

- Found in src/VoltageManager.sol [Line: 53](src/VoltageManager.sol#L53)

	```solidity
	        _ownerAddress = ownerAddress;
	```

- Found in src/VoltageManager.sol [Line: 67](src/VoltageManager.sol#L67)

	```solidity
	        _ownerAddress = newOwnerAddress;
	```

### [NC-6] Unused functions marked as `public` should be mared as `external`

This will increase readability and save gas.

- Found in src/AiArenaHelper.sol [Line: 162](src/AiArenaHelper.sol#L162)

	```solidity
	    function deleteAttributeProbabilities(uint8 generation) public {
	```

- Found in src/FighterFarm.sol [Line: 313](src/FighterFarm.sol#L313)

	```solidity
	    function mintFromMergingPool(
	```

- Found in src/FighterFarm.sol [Line: 338](src/FighterFarm.sol#L338)

	```solidity
	    function transferFrom(
	```

- Found in src/FighterFarm.sol [Line: 355](src/FighterFarm.sol#L355)

	```solidity
	    function safeTransferFrom(
	```

- Found in src/FighterFarm.sol [Line: 370](src/FighterFarm.sol#L370)

	```solidity
	    function reRoll(uint8 tokenId, uint8 fighterType) public {
	```

- Found in src/FighterFarm.sol [Line: 395](src/FighterFarm.sol#L395)

	```solidity
	    function contractURI() public pure returns (string memory) {
	```

- Found in src/FighterFarm.sol [Line: 402](src/FighterFarm.sol#L402)

	```solidity
	    function tokenURI(uint256 tokenId) public view override(ERC721) returns (string memory) {
	```

- Found in src/FighterFarm.sol [Line: 410](src/FighterFarm.sol#L410)

	```solidity
	    function supportsInterface(bytes4 _interfaceId)
	```

- Found in src/FighterFarm.sol [Line: 421](src/FighterFarm.sol#L421)

	```solidity
	    function getAllFighterInfo(
	```

- Found in src/FighterOps.sol [Line: 58](src/FighterOps.sol#L58)

	```solidity
	    function fighterCreatedEmitter(
	```

- Found in src/FighterOps.sol [Line: 84](src/FighterOps.sol#L84)

	```solidity
	    function viewFighterInfo(
	```

- Found in src/GameItems.sol [Line: 189](src/GameItems.sol#L189)

	```solidity
	    function setAllowedBurningAddresses(address newBurningAddress) public {
	```

- Found in src/GameItems.sol [Line: 212](src/GameItems.sol#L212)

	```solidity
	    function createGameItem(
	```

- Found in src/GameItems.sol [Line: 246](src/GameItems.sol#L246)

	```solidity
	    function burn(address account, uint256 tokenId, uint256 amount) public {
	```

- Found in src/GameItems.sol [Line: 253](src/GameItems.sol#L253)

	```solidity
	    function contractURI() public pure returns (string memory) {
	```

- Found in src/GameItems.sol [Line: 260](src/GameItems.sol#L260)

	```solidity
	    function uri(uint256 tokenId) public view override returns (string memory) {
	```

- Found in src/GameItems.sol [Line: 272](src/GameItems.sol#L272)

	```solidity
	    function getAllowanceRemaining(address owner, uint256 tokenId) public view returns (uint256) {
	```

- Found in src/GameItems.sol [Line: 283](src/GameItems.sol#L283)

	```solidity
	    function remainingSupply(uint256 tokenId) public view returns (uint256) {
	```

- Found in src/GameItems.sol [Line: 289](src/GameItems.sol#L289)

	```solidity
	    function uniqueTokensOutstanding() public view returns (uint256) {
	```

- Found in src/GameItems.sol [Line: 295](src/GameItems.sol#L295)

	```solidity
	    function safeTransferFrom(
	```

- Found in src/MergingPool.sol [Line: 201](src/MergingPool.sol#L201)

	```solidity
	    function addPoints(uint256 tokenId, uint256 points) public {
	```

- Found in src/MergingPool.sol [Line: 211](src/MergingPool.sol#L211)

	```solidity
	    function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
	```

- Found in src/Neuron.sol [Line: 158](src/Neuron.sol#L158)

	```solidity
	    function mint(address to, uint256 amount) public virtual {
	```

- Found in src/Neuron.sol [Line: 166](src/Neuron.sol#L166)

	```solidity
	    function burn(uint256 amount) public virtual {
	```

- Found in src/Neuron.sol [Line: 175](src/Neuron.sol#L175)

	```solidity
	    function approveSpender(address account, uint256 amount) public {
	```

- Found in src/Neuron.sol [Line: 188](src/Neuron.sol#L188)

	```solidity
	    function approveStaker(address owner, address spender, uint256 amount) public {
	```

- Found in src/Neuron.sol [Line: 200](src/Neuron.sol#L200)

	```solidity
	    function burnFrom(address account, uint256 amount) public virtual {
	```

- Found in src/RankedBattle.sol [Line: 388](src/RankedBattle.sol#L388)

	```solidity
	    function getUnclaimedNRN(address claimer) public view returns(uint256) {
	```

- Found in src/VoltageManager.sol [Line: 94](src/VoltageManager.sol#L94)

	```solidity
	    function useVoltageBattery() public {
	```

- Found in src/VoltageManager.sol [Line: 107](src/VoltageManager.sol#L107)

	```solidity
	    function spendVoltage(address spender, uint8 voltageSpent) public {
	```

### Time spent:
15 hours