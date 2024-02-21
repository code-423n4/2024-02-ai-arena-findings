# AI Arena - QA Report


| Id            | Issue                                                                                                         |
|---------------|---------------------------------------------------------------------------------------------------------------|
| [L-1](#l-1)   | Consider two-step confirmation for ownership transfer                                                         |
| [L-2](#l-2)   | `Verification::verify()` allows malleable (non-unique) signature                                              |
| [L-3](#l-3)   | `Verification::verify()` can return `true` for invalid signatures                                             |
| [L-4](#l-4)   | `FighterFarm::_delegatedAddress` cannot be updated                                                            |
| [L-5](#l-5)   | Neuron allowance spend during burn is inconsistent with OZ ERC20 being extended                               |
| [L-6](#l-6)   | Incrementing roundId in pickWinner allows ending rounds accidentally                                          |
| [L-7](#l-7)   | Players are not allowed to claim only a portion of their reward                                               |
| [L-8](#l-8)   | Inconsistent array lengths unnecessarily penalize the Player                                                  |
| [L-9](#l-9)   | Player can set invalid customAttributes which is undesirable                                                  |
| [L-10](#l-10) | Fighter point retrieval will be unable to access higher order Fighters                                        |
| [L-11](#l-11) | Iterating through all rounds means Fighter NFTs created after a certain round number can never claim a reward |
| [L-12](#l-12) | Allowing mid-round NRN Distribution alteration is unfair to Players                                           |
| [L-13](#l-13) | Allowing mid-round bpsLostPerLoss alteration is unfair to Players                                             |
| [NC-1](#nc-1) | Players can waste their own voltage                                                                           |
| [NC-2](#nc-2) | Transfer ownership leaves the initial owner as an admin role                                                  |
| [NC-3](#nc-3) | Attribute probabilities can exceed 100%                                                                       |
| [NC-4](#nc-4) | Incorrect @notice for fighterPoints                                                                           |
| [NC-5](#nc-5) | Deleting attributes will cause future Fighters to have invalid physical attributes                            |
| [NC-6](#nc-6) | Players can time their GameItems minting to achieve twice the allowance in 24hrs                              |
| [NC-7](#nc-7) | A Fighter cannot be staked on the round of transfer                                                           |
| [NC-8](#nc-8) | Players can be denied their NRN rewards when the mint cap is exceeded                                         |



## Low Risk 

### L-1
#### Consider two-step confirmation for ownership transfer
Ownership and all associated powers can be permanently lost with a copy/paste error or simple typo.

Instances: 7
[src/AiArenaHelper.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L61-L64)
[src/FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L120-L123)
[src/GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L108-L111)
[src/MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L89-L92)
[src/Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85-L88)
[src/RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L167-L170)
[src/VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L64-L67)

#### Recommended Mitigation
Implement a two-step procedure for ownership transfer, where the recipient is set as pending, and must 'accept' the assignment by making an affirmation transaction.
An available library implementation is Open Zeppelin's [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)



### L-2
#### `Verification::verify()` allows malleable (non-unique) signature
The signature verification used during the `FighterFarm::claimFighters` allows malleable (non-unique) signatures, as defined by EIP-2.

##### Recommended Mitigation
Use Open Zeppelin's [ECDSA::tryRecover()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/692dbc560f48b2a5160e6e4f78302bb93314cd88/contracts/utils/cryptography/ECDSA.sol#L128C57-L128C69) or replicate the enforcement of signatures being in the lower order.



### L-3
#### `Verification::verify()` can return `true` for invalid signatures
The signature verification used during the `FighterFarm::claimFighters` will allows invalid signatures when `FighterFarm::_delegatedAddress` is `address(0)`.

As an invalid signature will cause the `ecrecover` precompile to return `address(0)`, when `delegatedAddress` is also `address(0)` it would match,
allowing the caller the ability to mint an infinite of FighterFarm NFTs.


##### Recommended Mitigation
Address zero check in [`Verification::verify()`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Verification.sol#L16)
```diff
        require(signature.length == 65, "invalid signature length");
+       require(signer != address(0), "invalid signer address");
```



### L-4
#### `FighterFarm::_delegatedAddress` cannot be updated
Unlike the other members of `FighterFarm`, `_delegatedAddress` lacks any setter or instantiate function to update the address.

The uses for `_delegatedAddress` are setting the tokenURI and signing the messages players send in `FighterFarm::claimFighters()` to mint Fighter NFTs.
If control of the address is compromised, and infinite number of Fighter NFTs could be minted, and despite still controlling the owner address, nothing can be done.

As  `_delegatedAddress` is not `immutable`, there's a fair chance the setter was simply overlooked.


##### Recommended Mitigation
Add the setter to [FighterFarm](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L184)
```diff
+    function setDelegateAddress(address delegatedAddress) external {
+        require(msg.sender == _ownerAddress);
+        _delegatedAddress = delegatedAddress;
+    }
```



### L-5
#### Neuron allowance spend during burn is inconsistent with OZ ERC20 being extended
In `Neuron::burn()` the allowance is always decremented by the amount burnt, while the Open Zeppelin ERC20 [spend allowance](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/96e5c0830a83b61197dd4e29df59cb56499600ef/contracts/token/ERC20/ERC20.sol#L305-L315) treats ` type(uint256).max` as unlimited (does not decrement).

The transfer functions of `Neuorn` will use the OZ behaviour, which is inconsistent with how the burn function will behave.

##### Recommended Mitigation
Replicate the OZ allowance behaviour in [Neuron::burn()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L201-L203)
```diff
-        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
        _burn(account, amount);
-        _approve(account, msg.sender, decreasedAllowance);
+   _spendAllowance(account, msg.sender, amount);
```



### L-6
#### Incrementing roundId in pickWinner allows ending rounds accidentally 
In `MergingPool::pickWinner()` it is assumed the given winners are for the current round, however as the last operation the `roundId` is incremented.
As on every call `roundId` will be distinct, the check on whether the winners are already for that round will always pass. 

If the admin roles submits multiple transactions, although intended for the same round they will all be successfully processed, 
awarding multiple claims for the same two winners.

##### Recommended Mitigation
Instead of assuming the winners are for current round, have the Admin explicitly define the round in [MergingPool::pickWinner()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L118)
```diff
-     function pickWinner(uint256[] calldata winners) external {
+     function pickWinner(uint256[] calldata winners, uint winnerRoundId) external {
        require(isAdmin[msg.sender]);
        require(winners.length == winnersPerPeriod, "Incorrect number of winners");
-       require(!isSelectionComplete[roundId], "Winners are already selected");
-       require(!isSelectionComplete[winnerRoundId], "Winners are already selected");
```



### L-7
#### Players are not allowed to claim only a portion of their rewards
A Player is allowed to choose when they claim their rewards, allowing the potential of winning multiple awards before claiming.

When a Player calls `MergingPool::claimRewards()` the size of the given array parameters must match the number of all rewards, 
otherwise the transaction fails with a EVN revert of `array out-of-bounds access`.

A Player is not allowed to claim only a portion of their rewards, a valid case when the Player is mindful of gas expenditure.

##### Recommended Mitigation
Add an early exit condition to the loop, to allow partial claim of rewards in [MergingPool::claimRewards()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L149-L163)
```diff
        uint32 lowerBound = numRoundsClaimed[msg.sender];
+       uint rewardsToClaim = modelURIs.length;        
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
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
+                    
+                    if(claimIndex == rewardsToClaim){
+                       currentRound = roundId;
+                       j = winnersLength;
+                    }
                }
            }
        }
```



### L-8
#### Inconsistent array lengths unnecessarily penalize the Player
`MergingPool::claimRewards()` has three arrays parameters, but lack any check to ensure their lengths are the same. 
When the length of the arrays are inconsistent the transaction will revert (due to `array out-of-bounds access`), 

Although the fault lay with the Player, exiting early will limit the gas penalty (inconvenience) they suffer.

##### Recommended Mitigation
Require the array parameters to have the same lengths in [MergingPool::claimRewards()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139-L145)
```diff
    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
        external 
    {
    +   require(modelURIs.length == modelTypes.length, "Misnatch array length, URI and Types");
    +   require(modelURIs.length == customAttributes.length, "Misnatch array length, URI and Attributes");
```



### L-9
#### Player can set invalid customAttributes which is undesirable
When the Player calls `MergingPool::claimReward()` to mint their reward of a Fighter NFT, and they can choose any values for their attributes of `element` and `weight`.

The GameServer has a limited range of acceptable values for the attributes, when they are outside the range the `element` and `weight` attributes are changed to the closest acceptable value for the GameServer calculations. 

Allowing the Player to enter arbitrary values, permits them the opportunity to have the on-chain representation not match the off-chain behaviour, 
which based on discussion with the AI Arena team member `@guppy` is considered undesirable.

##### Recommended Mitigation
Require the Player input to be within the acceptable ranges in [MergingPool::claimReward()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139-L145)
```diff
    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
        external 
    {
+        uint numberOfClaims = customAttributes.length;
+        for( uint i; i < numberOfClaims; i++) {
+            require(customAttributes[i][0] == 100 || customAttributes[i][0] < 4, "element attrbitue range [0,1,2]");
+            require(customAttributes[i][0] == 100 || customAttributes[i][1] > 64, "weight attrbitue must be at least 65");
+            require(customAttributes[i][0] == 100 || customAttributes[i][1] < 96, "weight attrbitue must be at most 95");
+        }      
```


### L-10
#### Fighter point retrieval will be unable to access higher order Fighters
When the number of Fighters increases substantially, the expense of calling `MergingPool::getFighterPoints()` to retrieve their points will force the use of the `maxId` cap due to gas cost.
Ids above `maxId` will be excluded, with a likely use case for `getFighterPoints()` being a part of picking the winner, exclusion would be unfair to those Players.

Even when not called by a contract, calls that use excessive gas will cost bandwidth and encounter timeout limits for JSON-RPC retrieval, due to overwhelming data volume.

##### Recommended Mitigation
Implement pagination support by adding a start index in addition to the existing end index in [MergingPool::getFighterPoints()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L205-L211)     
```diff
-    function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
-        for (uint256 i = 0; i < maxId; i++) {
+    function getFighterPoints(uint256 minId, uint256 maxId) public view returns(uint256[] memory) {
+       require(minId < maxId, "minId must be smaller than maxId");
+       for (uint256 i = minId; i < maxId; i++) {
```



### L-11
#### Iterating through all rounds means Fighter NFTs created after a certain round number can never claim a reward
`MergingPool::claimRewards()` iterates through every round since the last claim (starting at zero for a new Fighter) upto the current round,
checking whether the Fighter was a winner in each round.

The loop contains `SSTORE` and `SLOAD` operations, so ever when the Fighter has not won, the gas cost adds up, and round id of ~15,000, 
the transaction will exceed the block gas limit on Arbitum (30,000,000).

Minting a reward is a much more gas intensive operation than merely looping, with multiple rewards, the round id of failure will reduce considerably.

#### Test
Gas consumption for a new Fighter NFT winning a prize after a number of dead rounds (rounds they won no prize in)

| Winning Round Id | Gas cost    |
|------------------|-------------|
| 0                | 392,292     |
| 10               | 413,652     |
| 20               | 435,012     |
| 30               | 456,372     |
| 40               | 477,732     |
| 50               | 499,092     |
| 100              | 605,892     |
| 200              | 819,492     |
| 300              | 1,033,092   |
| 1,000            | 2,528,292   |
| 2,000            | 4,664,292   |
| 5,000            | 11,072,292  |
| 10,000           | 21,752,292  |
| 15,000           | 32,432,292  |

<details>
<summary>Code</summary>
Add the following test to [`MergingPool.t.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/test/MergingPool.t.sol#L255) altering `deadRoundCount` accordingly

Run with `forge test --match-test test_claimRewards_gas_usage -vv --gas-report`
```solidity
    function test_claimRewards_gas_usage() external {
        uint rewardCount = 1;
        uint deadRoundCount = 20;

        // Create the fist two Fighters
        _mintFromMergingPool(_DELEGATED_ADDRESS);
        _mintFromMergingPool(_ownerAddress);
        assertEq(_fighterFarmContract.ownerOf(0), _DELEGATED_ADDRESS);
        assertEq(_fighterFarmContract.ownerOf(1), _ownerAddress);
        assertEq(_fighterFarmContract.totalSupply(), 2);

        // Dead rounds - _ownerAddress is not a winner, _DELEGATED_ADDRESS owns tokenId 0
        uint256[] memory noWinners = new  uint256[](2);
        for(uint deadRound; deadRound < deadRoundCount; deadRound++){
            _mergingPoolContract.pickWinner(noWinners);
        }

        // _ownerAddress is a winner
        uint256[] memory winners = _twoWinners();
        _mergingPoolContract.pickWinner(winners);
        assertEq(_mergingPoolContract.getUnclaimedRewards(_ownerAddress), rewardCount);

        string[] memory modelURIs = _createModelURIs(rewardCount);
        string[] memory modelTypes = _createModelTypes(rewardCount);
        uint256[2][] memory customAttributes = _createCustomAttributes(rewardCount);
        _mergingPoolContract.claimRewards(modelURIs, modelTypes, customAttributes);
        assertEq(_mergingPoolContract.getUnclaimedRewards(_ownerAddress), 0);
    }
```
</details>

##### Recommended Mitigation
Allow partial processing of the rounds, add an optional parameter for early iteration termination in [MergingPool::claimPrize()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139-L149)

```diff
+    /// @param uptoRoundId process claims upto the given round id, with zero meaning process upto the current round id.
    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
-       uint256[2][] calldata customAttributes
+       uint256[2][] calldata customAttributes,
+       uint256 uptoRoundId;        
    ) 
        external 
    {
+       require( uptoRoundId <= roundId, "uptoRoundId must not exceed roundId");    
+       require( uptoRoundId > numRoundsClaimed[msg.sender], "must process more than zero rounds");
        uint256 winnersLength;
        uint32 claimIndex = 0;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        uint256 upperBound = uptoRoundId == 0 ? roundId : uptoRoundId;
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
```



### L-12
#### Allowing mid-round NRN Distribution alteration is unfair to Players
One aspect the battle rewards for a Player is their share of the NRN distribution. 
The share being their portion of the points accrued in the round, with the points a function of their Fighters ELO factor and the stake.

`RankedBattle::setRankedNrnDistribution()` enables any admin role to change the NRN distribution for the current round, 
irrespective of whether any battles have already occurred and may be done multiple times a round.

The NRN distribution is part of the economic incentive for the Player, and the direct impact the Player has on points is from staking on their Fighter.
It is fair to assume a Player will use the NRN distribution as a factor in deciding their Fighter staking position. 
Allowing that variable to be altered mid-round is unfair to those who have already adjusted their stakes and had battles, as the circumstances of the points
landscape have change i.e. with the new NRN distribution the Player would have placed their stakes differently.

##### Recommended Mitigation
Ensure no battles have been recorded before the NRN distribution is updated in [RankedBattle::setRankedNrnDistribution()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L218-L221)
```diff
    function setRankedNrnDistribution(uint256 newDistribution) external {
        require(isAdmin[msg.sender]);
+       require(totalAccumulatedPoints[roundId] > 0);              
        rankedNrnDistribution[roundId] = newDistribution * 10**18;
    }
```



### L-13
#### Allowing mid-round bpsLostPerLoss alteration is unfair to Players
When a Fighter's battle record is update with a loss, and they have zero points, a portion of the stake is put at risk
(the amount is a function of stake and bpsLostPerLoss).
At risk stake can be won back, but otherwise at the end of the round, it is taken as protocol revenue.

`RankedBattle::setBpsLostPerLoss()` enables any admin role to change the bpsLostPerLoss for the current round,
irrespective of whether any battles have already occurred and may be done multiple times a round.

Allowing the bpsLostPerLoss update when battles may have already put stake at risk, is unfair to the Players and potentially also the protocol.
An increase to bpsLostPerLoss means the initial battles are given advantage as they put too less stake at risk, while a decrease places them at a disadvantage relative to later battles.

##### Recommended Mitigation
Ensure no battles have been recorded before the bpsLostPerLoss is updated in [RankedBattle::setBpsLostPerLoss()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L226-L229)
```diff
    function setBpsLostPerLoss(uint256 bpsLostPerLoss_) external {
        require(isAdmin[msg.sender]);
+       require(totalAccumulatedPoints[roundId] > 0);   
        bpsLostPerLoss = bpsLostPerLoss_;
    }
```



## Non-Critical

### NC-1
#### Players can waste their own voltage
 `VoltageManager::spendVoltage()` allows the `msg.sender` (the player) to spend (burn) their own voltage with no gain, unlike when `RankedBattle` spends it on their behalf.



### NC-2
#### Transfer ownership leaves the initial owner as an admin role
In the constructor, beside the `owner` role, the owner address is also assigned as the `admin` role, however `transferOwnership()` does not remove the admin role. 
After transferring ownership, the initial owner will remain as an admin.

Instances: 5

[src/GameItems.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L108-L111)
[src/MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L89-L92)
[src/Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85-L88)
[src/RankedBattle.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L167-L170)
[src/VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L64-L67)



### NC-3
#### Attribute probabilities can exceed 100%
An absence of a sum check would allow the attribute matrix to exceed the usable range of `0-99` in [AiArenaHelper::addAttributeProbabilities()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L131-L139)



### NC-4
#### Incorrect @notice for fighterPoints
The `@notice` comment incorrectly describes `tokenIds` (Id of the Fighter NFT) as `address` in [src/MergingPool.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L50)

```diff
+    /// @notice Mapping of tokenId to fighter points.
-    /// @notice Mapping of address to fighter points.
    mapping(uint256 => uint256) public fighterPoints;
```



### NC-5
#### Deleting attributes will cause future Fighters to have invalid physical attributes
`AiArenaHelper::deleteAttributeProbabilities()` allows an admin to delete the `attributeProbabilities` for a generation of Fighters.
When there are no `attributeProbabilities` during creating a Fighter, then all the physical are set as index 0,
while when present the physical attribute indices begin at 1.



### NC-6
#### Players can time their GameItems minting to achieve twice the allowance in 24hrs
`GameItems` imposes daily allowance of item minting on Players (for economic, rather game balance purposes).

Using getter to inspect their allowance refresh time, a Player can mint the maximum allowance of GameItems just prior to their daily allowance refresh time,
and then again after. Achieving twice their daily allowance, within one day, technically breaking the invariant of a daily allowance.



### NC-7
#### A Fighter cannot be staked on the round of transfer
If a Player has removed stake on their Fighter with `RankedBattle::unstakeNRN()`, they cannot in the same round add stake to that Fighter.

When a Player removes their stake on a Fighter to transfer to another Player, that new Player cannot stake on their newly acquire Fighter until the next round.

Unexpected behaviour as the new Player had not unstaked that Fighter, but as the semaphore flagging applies only to the Fighter Id, irrespective of the owner.



### NC-8
#### Players can be denied their NRN rewards when the mint cap is exceeded
Rewards given by `RankedBattle::claimNRN()` are minted from `Neuron.sol`, which has a mint cap.

If the amount of NRN to mint for the rewards exceed the mint cap, the Players will be denied their rewards.

