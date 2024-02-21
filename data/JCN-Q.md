## Findings Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-users-can-prevent-losses-for-ranked-battles-that-they-initiated) | Users can prevent losses for ranked battles that they initiated | Low |
| [L-02](#l-02-users-can-avoid-the-economic-effects-of-losses-at-least-once-per-round) | Users can avoid the economic effects of losses at least once per round | Low |
| [L-03](#l-03-users-with-mintpasses-can-ensure-that-they-only-mint-dendroids-which-are-more-valuable-than-champions) | Users with mintpasses can ensure that they only mint dendroids, which are more valuable than champions | Low |
| [L-04](#l-04-signature-can-be-reused-if-the-game-is-deployed-on-another-chain) | Signature can be reused if the game is deployed on another chain | Low |

**Note to Lookout and Judge**: 

Issues `L-01` and `L-02` are dependent on "frontrunning" capabilities. I understand that Arbitrum will be the initial chain in which this game will launch (frontrunning is not currently feasible), however the sponsors have stated in public channels that their decision to launch on another chain is `undecided`. That being said, frontrunning *may* be a possibility in the future if they choose to launch on a chain where that action is feasible. For this reason I have chosen to submit these issues so that the sponsors could be made aware of potential vulnerabilities that could arise if they choose to launch on such a chain. I am placing this disclaimer here in the event that the information I just provided makes these issues invalid (exploit currently infeasible on initial chain), in which case the Lookout/Judge would not need to spend time reading these issues in their entirety.

## [L-01] Users can prevent losses for ranked battles that they initiated

### Bug Description
When two players enter into a ranked battle one of the players must initiate the battle. The player who initiates the battle must spend voltage in order to do so. This "spending" of voltage actually occurs *after* the battle occurs (off-chain) and is enforced when the battle results are published on-chain via `RankedBattle::updateBattleRecord`.

[RankedBattle::updateBattleRecord](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L322-L347)

```solidity
322:    function updateBattleRecord(
323:        uint256 tokenId, 
324:        uint256 mergingPortion,
325:        uint8 battleResult,
326:        uint256 eloFactor,
327:        bool initiatorBool
328:    ) 
329:        external 
330:    {   
331:        require(msg.sender == _gameServerAddress);
332:        require(mergingPortion <= 100);
333:        address fighterOwner = _fighterFarmInstance.ownerOf(tokenId);
334:        require(
335:            !initiatorBool ||
336:            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
337:            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST // @audit: ensures the initiator has enough voltage to spend
338:        );
...
345:        if (initiatorBool) {
346:            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST); // @audit: voltage spent here
347:        }
```

As we can see above, the require statement on lines 334 - 338 will check if the `fighterOwner` was the initiator (i.e. `initiatorBool == true`) and if so the `ownerVoltageReplenishTime[initiator]` and `ownerVoltage[initiator]` mappings will be enforced to be within a specific bounds. If this require statement passes, then this means that the appropriate amount of the initiator's voltage can be spent, as shown on line 346 via `VoltageManager::spendVoltage`.

[VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L105-L120)

```solidity
105:    function spendVoltage(address spender, uint8 voltageSpent) public {
106:        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]); // @audit: users can spend their own voltage
107:        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
108:            _replenishVoltage(spender);
109:        }
110:        ownerVoltage[spender] -= voltageSpent; 
111:        emit VoltageRemaining(spender, ownerVoltage[spender]);
112:    }
113:
114:    /// @notice Replenishes voltage and sets the replenish time to 1 day from now
115:    /// @dev This function is called internally to replenish the voltage for the owner.
116:    /// @param owner The address of the owner
117:    function _replenishVoltage(address owner) private {
118:        ownerVoltage[owner] = 100;
119:        ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days); 
120:    }       
```

However, as shown above the `spendVoltage` function allows the initiator to "spend" their own voltage at any given time. This allows the following scenario to occur:

1. Initiator loses a ranked battle
2. Initiator invokes a call to `spendVoltage(initiator, ownerVoltage[initiator])` before the `updateBattleRecord` call and depletes all their voltage
3. The `ownerVoltage[initiator]` mapping is set to 0 and the `ownerVoltageReplenishTime[initiator]` mapping is set to `block.timestamp + 1 days`
4. The require statement in the `updateBattleRecord` call reverts since `ownerVoltageReplenishTime[initiator] > block.timestamp` and `ownerVoltage[initiator] < VOLTAGE_COST`

The above scenario will result in no loss being recorded for the initiator.

### Impact
A user can prevent losses (from ranked battles they initiated) from being recorded on-chain by frontrunning the `updateBattleRecord` call with a call to `spendVoltage`. The user will be able to avoid losing points and/or having a portion of their staked `NRN` being confiscated when they lose ranked battles that they initiated.

Note that after this action is performed the user must either purchase a `battery` to manually replenish their voltage, or wait `1 day` for their voltage to automatically replenish before they are able to initiate another ranked battle.

### Proof of Concept

Place the following test inside of `/test/StakeAtRisk.t.sol` and run with `forge --mc StakeAtRiskTest --mt testSpendVoltagePreventLoss`:

```solidity
    function testSpendVoltagePreventLoss() public {
        address player = vm.addr(3);
        uint256 stakeAmount = 3_000 * 10 ** 18;
        uint256 expectedStakeAtRiskAmount = (stakeAmount * 100) / 100000;
        uint256 tokenId = 0;

        // player gets fighter
        _mintFromMergingPool(player);
        assertEq(_fighterFarmContract.ownerOf(tokenId), player);

        // player gets NRN
        _fundUserWith4kNeuronByTreasury(player);
        vm.prank(player);
        // player stakes NRN
        _rankedBattleContract.stakeNRN(stakeAmount, 0);
        assertEq(_rankedBattleContract.amountStaked(0), stakeAmount);

        // player battles (off-chain)

        // player spends their own voltage (front-run `updateBattleRecord` call)
        vm.prank(player);
        _voltageManagerContract.spendVoltage(player, 100);

        // loses battle and game server contract attempts to push results on-chain
        vm.startPrank(address(_GAME_SERVER_ADDRESS));
        vm.expectRevert();
        _rankedBattleContract.updateBattleRecord(0, 50, 2, 1500, true);
        vm.stopPrank();
        assertEq(_stakeAtRiskContract.stakeAtRisk(0, 0), 0); // player has no stake at risk
        assertEq(_stakeAtRiskContract.getStakeAtRisk(tokenId), 0); // player has no stake at risk
    }
```

### Recommended Mitigation Steps
Unless there is a specific use case for it, a potential mitigation would be to *not* allow users to spend their own voltage. There is already a function for a user to manually increase their voltage via a `battery` and their voltage is replenished and then depleted by the appropriate amount during the battle record updates. 

## [L-02] Users can avoid the economic effects of losses at least once per round

## Bug Description
Users have the option of staking `NRN` prior to participating in ranked battles and this will allow them to participate in the game economy. If the user then wins a ranked battle they will acquire points, which will make them eligible to claim rewards when a new round starts. However, if a user loses a ranked battle they are expected to incur the effects of the loss by either:

1. Having their points deducted (if they had points)
2. Having a portion of their staked `NRN` confiscated and thus placed `at risk` of being lost (if they did not have points)

After a ranked battle the results will be published in the `RankedBattle` contract via a permissioned call to `updateBattleRecord`.

[RankedBattle::updateBattleRecord](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L341-L344)

```solidity
341:        uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
342:        if (amountStaked[tokenId] + stakeAtRisk > 0) {
343:            _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
344:        }
```

As we can see above, if user currently has `NRN` staked or if they currently have stake that is `at risk`, then the `_addResultsPoints` function will be invoked. If the user lost the ranked battle, this function will either decrease the user's points or confiscate a portion of the user's stake and transfer it to the `StakeAtRisk` contract:

[RankedBattle::_addResultPoints](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L479-L497)

```solidity
479:            if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
480:                /// If the fighter has a positive point balance for this round, deduct points 
481:                points = stakingFactor[tokenId] * eloFactor;
482:                if (points > accumulatedPointsPerFighter[tokenId][roundId]) {
483:                    points = accumulatedPointsPerFighter[tokenId][roundId];
484:                }
485:                accumulatedPointsPerFighter[tokenId][roundId] -= points;
486:                accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
487:                totalAccumulatedPoints[roundId] -= points;
488:                if (points > 0) {
489:                    emit PointsChanged(tokenId, points, false);
490:                }
491:            } else {
492:                /// If the fighter does not have any points for this round, NRNs become at risk of being lost
493:                bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
494:                if (success) {
495:                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
496:                    amountStaked[tokenId] -= curStakeAtRisk;
497:                }
```

A user is able to avoid the effects of the above code if they call `unstakeNRN` before the `updateBattleRecord` call. 

[RankedBattle::unstakeNRN](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L270-L275)

```solidity
270:    function unstakeNRN(uint256 amount, uint256 tokenId) external {
271:        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
272:        if (amount > amountStaked[tokenId]) {
273:            amount = amountStaked[tokenId];
274:        }
275:        amountStaked[tokenId] -= amount;
```

As we can see above a user is able to unstake at any time and thus can invoke this function to make their `amountStaked[user]` equal to 0 immediately before the `updateBattleRecord` call. Provided that they do not have any stake currently `at risk`, i.e. `stakeAtRisk == 0`, this would cause the `_addResultPoints` function on line 343 of the `updateBattleRecord` function to *not* be invoked. As a reminder:

[RankedBattle::updateBattleRecord](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L341-L344)

```solidity
341:        uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
342:        if (amountStaked[tokenId] + stakeAtRisk > 0) { // @audit: after unstake `amountStaked` + `stakeAtRisk` == 0
343:            _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
344:        }
```

This will result in the user avoiding the economic effects of their loss, which are enforced in the `_addResultPoints` internal function.

Note that due to [this line](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L248) in the `stakeNRN` function, a user will not be able to re-stake behind their fighter during the same round if they already unstaked any amount.

## Impact
A user can avoid the effects of losses (points deduction/stake at risk) once per round by unstaking their `NRN` immediately before the `updateBattleRecord` call. Although a user is able to avoid the negative effects of incurring this loss, this action will prevent the user from participating in ranked battles for the rest of the round. Therefore, this exploit would only be beneficial to a user if it was strategically/economically favorable for them to avoid this single loss (i.e. they could have been on a "win streak", or did not want to lose any of their stake).

## Proof of Concept

Place the following test inside of `/test/StakeAtRisk.t.sol` and run with `forge --mc StakeAtRiskTest --mt testUnstakeToPreventStakeAtRisk`:

```solidity
    function testUnstakeToPreventStakeAtRisk() public {
        address player = vm.addr(3);
        uint256 stakeAmount = 3_000 * 10 ** 18;
        uint256 expectedStakeAtRiskAmount = (stakeAmount * 100) / 100000;
        uint256 tokenId = 0;

        // player gets fighter
        _mintFromMergingPool(player);
        assertEq(_fighterFarmContract.ownerOf(tokenId), player);

        // player gets NRN
        _fundUserWith4kNeuronByTreasury(player);
        vm.prank(player);
        // player stakes NRN
        _rankedBattleContract.stakeNRN(stakeAmount, 0);
        assertEq(_rankedBattleContract.amountStaked(0), stakeAmount);

        // player battles (off-chain)

        // player unstakes their NRN (front-run `updateBattleRecord` call)
        vm.prank(player);
        _rankedBattleContract.unstakeNRN(stakeAmount, 0);
        assertEq(_rankedBattleContract.amountStaked(0), 0);

        // loses battle and game server contract attempts to confiscate a portion of their stake that is now at risk
        vm.prank(address(_GAME_SERVER_ADDRESS));
        _rankedBattleContract.updateBattleRecord(0, 50, 2, 1500, true);
        assertEq(_stakeAtRiskContract.stakeAtRisk(0, 0), 0); // player has no stake at risk
        assertEq(_stakeAtRiskContract.getStakeAtRisk(tokenId), 0); // player has no stake at risk
    }
```

## Recommended Mitigation Steps
A potential mitigation would be to restrict users from calling `unstakeNRN` when they enter into a ranked battle (this would require comms between off-chain infra). This will prevent users from updating their stake after the ranked battle has occurred, i.e. *after* the user participates in the ranked battle off-chain and *before* the `RankedBattle::updateBattleRecord` function is invoked on-chain.

## [L-03] Users with `mintpasses` can ensure that they only mint `dendroids`, which are more valuable than `champions`

### Bug Description
The sponsors have indicated that `dendroids` are more valuable that `champions` and that user's who have `mintpasses` should only be allowed to mint a `dendroid` if their `mintpass` metadata specifies a `dendroid` fighter type. A fighter is determined to be a `dendroid` if the `fighterType` of the fighter is equal to `1`:

[FighterFarm::_createNewFighter](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L509-L528)

```solidity
509:        bool dendroidBool = fighterType == 1;
510:        FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(
511:            newDna,
512:            generation[fighterType],
513:            iconsType,
514:            dendroidBool 
515:        );
516:        fighters.push(
517:            FighterOps.Fighter( // @audit: struct that defines fighter NFT
518:                weight,
519:                element,
520:                attrs,
521:                newId,
522:                modelHash,
523:                modelType,
524:                generation[fighterType],
525:                iconsType,
526:                dendroidBool // @audit: if true -> dendroid
527:            )
528:        );
```

User's are able to redeem their `mintpasses` via the `FighterFarm::redeemMintPass` function. This function only validates the `mintpassIdsToBurn` (tokenId of `mintpass`). The value of the `fighterType`, and all other calldata input values, are *not* validated:

[FighterFarm::redeemMintPass](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L233-L260)

```solidity
233:    function redeemMintPass(
234:        uint256[] calldata mintpassIdsToBurn,
235:        uint8[] calldata fighterTypes, // @audit: only length is validated
236:        uint8[] calldata iconsTypes, // @audit: not validated 
237:        string[] calldata mintPassDnas, // @audit: only length is validated
238:        string[] calldata modelHashes, // @audit: only length is validated
239:        string[] calldata modelTypes // @audit: only length is validated
240:    ) 
241:        external 
242:    {
243:        require(
244:            mintpassIdsToBurn.length == mintPassDnas.length && 
245:            mintPassDnas.length == fighterTypes.length && 
246:            fighterTypes.length == modelHashes.length &&
247:            modelHashes.length == modelTypes.length
248:        );
249:        for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
250:            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
251:            _mintpassInstance.burn(mintpassIdsToBurn[i]);
252:            _createNewFighter(
253:                msg.sender, 
254:                uint256(keccak256(abi.encode(mintPassDnas[i]))), 
255:                modelHashes[i], 
256:                modelTypes[i],
257:                fighterTypes[i], 
258:                iconsTypes[i],
259:                [uint256(100), uint256(100)]
260:            );
```

As we can see above, a user is able to pass arbitrary values for any of the calldata parameters for this function, with the exception of the `mintpassIdsToBurn`. Specifically, this allows the user to pass in a `fighterType == 1`, which will guarantee that the user will mint a `dendroid`, even if the metadata of their `mintPass` specifies a `champion`.

### Impact
Any user with a `mintPass` can ensure that the fighter(s) they mint via `redeemMintPass` will be a `dendroid`, and therefore have the greatest value, regardless of what the metadata of their `mintPass` specifies.

### Recommended Mitigation Steps
All calldata inputs should be validated to ensure that a user is only able to mint the correct fighter type with the appropriate attributes.

## [L-04] Signature can be reused if the game is deployed on another chain

### Bug Description
When a user mints a fighter via `FighterFarm::claimFighters`, the user must supply a signature from the server. The signature digest is made up of the user's address (msg.sender), values that dictate the fighter type (`numToMint`), and state that pertains to the user (`nftsClaimed`):

[FighterFarm::claimFighters](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L199-L220)

```solidity
199:        bytes32 msgHash = bytes32(keccak256(abi.encode(
200:            msg.sender, // @audit: user's address
201:            numToMint[0],  // @audit: calldata values (supplied by user)
202:            numToMint[1],
203:            nftsClaimed[msg.sender][0], // @audit: state that changes after mint, starts at 0
204:            nftsClaimed[msg.sender][1]
205:        )));
206:        require(Verification.verify(msgHash, signature, _delegatedAddress));
207:        uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
208:        require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
209:        nftsClaimed[msg.sender][0] += numToMint[0];
210:        nftsClaimed[msg.sender][1] += numToMint[1];
211:        for (uint16 i = 0; i < totalToMint; i++) {
212:            _createNewFighter(
213:                msg.sender, 
214:                uint256(keccak256(abi.encode(msg.sender, fighters.length))),
215:                modelHashes[i], 
216:                modelTypes[i],
217:                i < numToMint[0] ? 0 : 1,
218:                0,
219:                [uint256(100), uint256(100)]
220:            );
```
The `msg.sender` can be the same across different chains, the `numToMint` values are supplied by the user, and the `nftsClaimed` mapping is initialized to 0, meaning this mapping would be the same for the first mints on two different chains. Additionally, a `deadline` or `chainId` is not part of the signature digest.

Therefore, a user is able to acquire this signature via the front-end, use it to mint via the above function on the initial chain, and then reuse the signature at a future time if the game is deployed on a different chain.

### Impact
A user can potentially reuse server signatures in the future in order to mint free fighters on different chains that the game gets deployed on. Note that the assumptions here are that the `_delegatedAddress` will be the same across all the chains and that the game is deployed on another chain to begin with.

### Recommended Mitigation Steps
If it is a possibility that this game may be deployed on another chain, I would suggest protecting against signature replay attacks by explicitly incorporating a `deadline` and `chainId` as part of the signature digest and then validating these values.