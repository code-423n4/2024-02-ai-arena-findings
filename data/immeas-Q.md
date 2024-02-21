# QA Report


## Summary

| id | title |
| --- | --- |
| [L-01](#l-01-player-can-use-voltage-battery-when-they-are-at-full-voltage) | player can use voltage battery when they are at full voltage |
| [L-02](#l-02-user-redeeming-mint-pass-can-chose-fighter-type) | user redeeming mint pass can chose fighter type |
| [L-03](#l-03-no-address0-validation-for-_delegateaddress) | no `address(0)` validation for `_delegateAddress` |
| [L-04](#l-04-seller-of-fighter-can-grief-and-make-fighter-unable-to-compete-in-current-round) | seller of fighter can grief and make fighter unable to compete in current round |
| [L-05](#l-05-address-of-winner-is-stored-when-winner-is-picked) | address of winner is stored when winner is picked |
| [L-06](#l-06-changing-_stakeatriskinstance-in-rankedbattle-will-lock-players-nrn) | changing `_stakeAtRiskInstance` in `RankedBattle` will lock players `NRN` |
| [L-07](#l-07-changing-_neuroninstance-in-rankedbattle-will-mess-up-accounting) | changing `_neuronInstance` in `RankedBattle` will mess up accounting |
| [L-08](#l-08-roles-in-neuron-cannot-be-revoked) | roles in Neuron cannot be revoked |
| [L-09](#l-09-allowedburningaddresses-cannot-be-removed) | `allowedBurningAddresses` cannot be removed |
| [L-10](#l-10-account-cannot-be-removed-from-hasstakerrole-in-fighterfarm) | account cannot be removed from `hasStakerRole` in `FighterFarm` |
| [L-11](#l-11-old-owner-isnt-removed-from-admin-when-owner-is-transferred) | old owner isn't removed from admin when owner is transferred |
| [L-12](#l-12-totalbattles-will-be-double-what-it-should-be) | `totalBattles` will be double what it should be |
| [L-13](#l-13-a-gameitems-dailyallowance-cannot-be-set-larger-than-65535) | a GameItems `dailyAllowance` cannot be set larger than `65535` |
| [L-14](#l-14-last-wei-of-nrn-cannot-be-minted) | last wei of `NRN` cannot be minted |
| [L-15](#l-15-winloss-combination-are-not-atomic) | win/loss combination are not atomic |
| [L-16](#l-16-fixed-amounts-can-lead-to-future-economic-issues) | fixed amounts can lead to future economic issues |
| [NC-01](#nc-01-some-fields-are-not-included-in-fighterfarmgetallfighterinfo) | some fields are not included in `FighterFarm::getAllFighterInfo` |
| [NC-02](#nc-02-fighterfarmmintfrommergingpool-uses-msgsender-which-always-is-mergingpool) | `FighterFarm::mintFromMergingPool` uses `msg.sender` which always is `MergingPool` |
| [NC-03](#nc-03-unnecessary-voltage-check-in-rankedbattle) | unnecessary voltage check in `RankedBattle` |
| [NC-04](#nc-04-lack-of-events-emitted) | Lack of events emitted |
| [NC-05](#nc-05-generation-is-uint8-in-field-but-uint16-in-return) | `generation` is `uint8` in field but `uint16` in return |
| [NC-06](#nc-06-indentation-is-off) | Indentation is off |
| [NC-07](#nc-07-loop-variable-currentround-is-confusing) | loop variable `currentRound` is confusing |
| [NC-08](#nc-08-usage-of-sensitive-words) | usage of sensitive words |
| [NC-09](#nc-09-some-test-assertions-overly-complicated) | some test assertions overly complicated |
| [NC-10](#nc-10-misspelled-test) | misspelled test |


## Low


### L-01 player can use voltage battery when they are at full voltage
When using a voltage battery a check is done to verify that a user isn't at full voltage:
[`VoltageManager::useVoltageBattery`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L93-L94):
```solidity
File: src/VoltageManager.sol

93:    function useVoltageBattery() public {
94:        require(ownerVoltage[msg.sender] < 100);
```

However, just because a user has a voltage less than `100` doesn't mean they aren't at full voltage. As voltage is only replenished when used. A user could have not spent voltage in more than a day and still use a battery. Even though they would be at full voltage as `ownerVoltageReplenishTime` would be greater than `block.timestamp`.

#### PoC
Test in `test/VoltageManager.t.sol`:
```solidity
    function testUseVoltageBatteryWhenFullVotage() public {
        address player = vm.addr(3);
        _mintGameItemForReceiver(player);

        vm.startPrank(player);
        _voltageManagerContract.spendVoltage(player, 10);

        // voltage is back to max
        vm.warp(block.timestamp + 1 days + 1);

        // voltage battery can still be used
        _voltageManagerContract.useVoltageBattery();
        vm.stopPrank();
    }
```

#### Recommendation
Consider in addition to checking that `voltage < 100` also check that `ownerVoltageReplenishTime` isn't passed.
```diff
-       require(ownerVoltage[msg.sender] < 100);
+       require(ownerVoltage[msg.sender] < 100 && ownerVoltageReplenishTime[msg.sender] > block.timestamp);
```


### L-02 user redeeming mint pass can chose fighter type
When redeeming a mint pass the redeemer calls [`FighterFarm::redeemMintPass`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L233-L235):
```solidity
File: src/FighterFarm.sol

233:    function redeemMintPass(
234:        uint256[] calldata mintpassIdsToBurn,
235:        uint8[] calldata fighterTypes,
```

This is then passed to [`FighterFarm::_createNewFighter`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L509):
```solidity
File: src/FighterFarm.sol

509:        bool dendroidBool = fighterType == 1;
```

Where it decides if the fighter is a dendroid or not.

According to [the documentation](https://docs.aiarena.io/gaming-competition/nft-makeup), dendroids are supposed to be rare and revealed with a story:
> 📺 **Dendroids** - Dendroids are a more exclusive class of NFTs. They have the ability to incorporate other metaverse assets (e.g. NFTs from other projects) into the Arena! These are very rare and more difficult to obtain. You’ll have to follow the story to find out how to get access to one of these!

However, anyone with a mintpass can chose to mint one.

#### PoC
Test in `test/FighterFarm.t.sol`:
```solidity
    function testRedeemMintPassDendroid() public {
        uint8[2] memory numToMint = [1, 0];
        bytes memory signature = abi.encodePacked(
            hex"20d5c3e5c6b1457ee95bb5ba0cbf35d70789bad27d94902c67ec738d18f665d84e316edf9b23c154054c7824bba508230449ee98970d7c8b25cc07f3918369481c"
        );
        string[] memory _tokenURIs = new string[](1);
        _tokenURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _mintPassContract.claimMintPass(numToMint, signature, _tokenURIs);
        
        // once owning one i can then redeem it for a fighter
        uint256[] memory _mintpassIdsToBurn = new uint256[](1);
        uint8[] memory _fighterTypes = new uint8[](1);
        uint8[] memory _iconsTypes = new uint8[](1);
        string[] memory _strings = new string[](1);

        _mintpassIdsToBurn[0] = 1;
        _strings[0] = "foo";
        // redeemer of mintpass can simply chose to mint a dendroid
        _fighterTypes[0] = 1;
        _iconsTypes[0] = 1;

        _mintPassContract.approve(address(_fighterFarmContract), 1);
        _fighterFarmContract.redeemMintPass(_mintpassIdsToBurn, _fighterTypes, _iconsTypes, _strings, _strings, _strings);

        // what is minted is a dendroid
        (, , , , , , , , bool dendroidBool) = _fighterFarmContract.fighters(0);
        assertTrue(dendroidBool);
    }
```

#### Recommendation
Consider if the user should be able to do this and possibly limit it to only regular fighters


### L-03 no `address(0)` validation for `_delegateAddress`
This is mentioned in the automated report but it fails to account for the full impact.

Since the `_delegatedAddress` is used as comparison in the signature verification, if the `_delegatedAddress` is `address(0)` any invalid signature will be accepted by `Verification.verify`:
[`FighterFarm::claimFighters`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L206):
```solidity
File: src/FighterFarm.sol

206:        require(Verification.verify(msgHash, signature, _delegatedAddress));
```

Hence if `_delegatedAddress` was accidentally set to `address(0)` anyone could mint fighters as soon as the contract is deployed.

#### PoC
Test in `test/FighterFarm.t.sol`:
```solidity
    function testDelegateAddress0() public {
        FighterFarm fighterFarm = new FighterFarm(
            address(this),
            address(0), // delegateAddress is 0
            _treasuryAddress
        );
        fighterFarm.instantiateAIArenaHelperContract(address(_helperContract));

        uint8[2] memory numToMint = [1, 0];
        string[] memory strings = new string[](1);
        strings[0] = "";

        // delegateAddress 0 means that any signature will pass
        bytes memory claimSignature = abi.encodePacked(
            hex"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
        );
        fighterFarm.claimFighters(numToMint, claimSignature, strings, strings);

        assertEq(fighterFarm.ownerOf(0),address(this));
    }
```

#### Recommendation
Consider validating that `_delegateAddress` is not set to `address(0)`


### L-04 seller of fighter can grief and make fighter unable to compete in current round
A seller of a fighter can grief the buyer by making the fighter unable to compete for points in the current round. By calling [`RankedBattle::unstakeNRN`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L282) before selling it:
```solidity
File: src/RankedBattle.sol

282:        hasUnstaked[tokenId][roundId] = true;
```
This would cause the fighter not to be able to compete for points as the buyer would have no way of staking `NRN`. The grief would only affect the current round though.

#### Recommendation
Consider checking that the fighter hasn't unstaked in the current round as well before allowing transfer.


### L-05 address of winner is stored when winner is picked
When a winner is picked in `MergingPool`, the winner address is stored in an array:
[`MergingPool::pickWinner`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L125):
```solidity
File: src/MergingPool.sol

125:            currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);
```

Then the winner can call `claimRewards` and have their merged fighter minted.

The issue is that the owner of the fighter when `pickWinner` is called can be anything, a contract that perhaps cannot call `claimRewards` (like a marketplace or vault). Even if the _actual_ owner later can transfer the fighter to their EOA, the address has already been stored, hence they would have no way of claiming their merged fighter.

#### Recommendation
Consider changing so that only the token in question is recorded to have won for a round. Then do the comparison to check owner when calling `claimRewards` instead.


### L-06 changing `_stakeAtRiskInstance` in `RankedBattle` will lock players `NRN`
In `RankedBattle` the bookkeeping of stake at risk is handled by the `StakeAtRisk` contract. Once a player loses and gets their stake put at risk, `NRN` are transferred to the `StakeAtRisk` contract. Where they can either win it back or lose it once the round is over.

The issue here is that the `_ownerAddress` can change the `_stakeAtRiskInstance` in `RankedBattle`, [`RankedBattle::setStakeAtRiskAddress`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L192-L196):
```solidity
File: src/RankedBattle.sol

192:    function setStakeAtRiskAddress(address stakeAtRiskAddress) external {
193:        require(msg.sender == _ownerAddress);
194:        _stakeAtRiskAddress = stakeAtRiskAddress;
195:        _stakeAtRiskInstance = StakeAtRisk(_stakeAtRiskAddress);
196:    }
```

If there is any current stake at risk this would be lost. Also, the current round would be very weird as the `roundId` in `StakeAtRisk` is only synced once a new round is started.

#### Recommendation
Consider removing the ability to change `_stakeAtRiskInstance` in `RankedBattle` after it has been set.


### L-07 changing `_neuronInstance` in `RankedBattle` will mess up accounting
`_neuronInstance` in `RankedBattle` is the address of the `NRN` token. There is a call to change this address, [`RankedBattle::instantiateNeuronContract`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L201-L204):
```solidity
File: src/RankedBattle.sol

201:    function instantiateNeuronContract(address nrnAddress) external {
202:        require(msg.sender == _ownerAddress);
203:        _neuronInstance = Neuron(nrnAddress);
204:    }
```

Were this to be done, all the old `NRN` in the contract would effectively be locked. And all the bookkeeping done in `RankedBattle` would be off. Hence it is a very dangerous state to change.

#### Recommendation
Consider removing the ability to change `_neuronInstance` in `RankedBattle` after it has been set or move the assigning of `_neuronInstance` to the constructor.


### L-08 roles in Neuron cannot be revoked

In `Neuron` there are a couple of proviliged roles, [`MINTER_ROLE`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L93-L96) which can mint `NRN`, [`STAKER_ROLE`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L101-L104) which can grant allowance between any two accounts and [`SPENDER_ROLE`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L109-L112) which can grant allowance from any account to `msg.sender`.

All these roles have very sensitive priviliges which if they were misused would cause mayhem.

The issue is that there is no way to revoke any of them. All the owner can do is grant them to accounts. Were they to fall into malicious hands the protocol wouldn't be able to do anything about it.

#### Recommendation
Consider adding a call to revoke a role as well


### L-09 `allowedBurningAddresses` cannot be removed

[`GameItems::setAllowedBurningAddresses`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L185-L188):
```solidity
File: src/GameItems.sol

185:    function setAllowedBurningAddresses(address newBurningAddress) public {
186:        require(isAdmin[msg.sender]);
187:        allowedBurningAddresses[newBurningAddress] = true;
188:    }
```

Here you can set an address to allow it to burn tokens. There is however no call to revoke this ability. If an account is compromised and starts to unwantedly burn too many tokens there's no way to stop it.

#### Recommendation
Consider adding a call to remove the ability to burn from an address or change the current call adding a bool that sets the status.


### L-10 account cannot be removed from `hasStakerRole` in `FighterFarm`
The `RankedBattle` contract has the `staker` role in `FighterFarm`, this is set by the call [`FighterFarm::addStaker`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L139-L142):
```solidity
File: src/FighterFarm.sol

139:    function addStaker(address newStaker) external {
140:        require(msg.sender == _ownerAddress);
141:        hasStakerRole[newStaker] = true;
142:    }
```

This allows a caller to change the staking flag which controls if a fighter is transferable or not.

There is however no call to remove this ability from a caller. Hence if this would be given to a compromised or malicious account there would be no way to remove it.

#### Recommendation
Consider adding a call to remove the staker role or change the current call adding a bool to set the status.


### L-11 old owner isn't removed from admin when owner is transferred
The `GameItems`, `MergingPool`, `Neuron`, `RankedBattle` and `VoltageManager` contracts all have an `owner` and an `admin` role. When created, in the constructor, the  `owner` is automatically given the `admin` role as well. All five contracts also have a call to transfer ownership.

[`GameItems::constructor`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L98) and [`GameItems::transferOwnership`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L108-L111) as an example, all of the five look the same:
```solidity
File: src/GameItems.sol

 89:        isAdmin[_ownerAddress] = true;
...
108:    function transferOwnership(address newOwnerAddress) external {
109:        require(msg.sender == _ownerAddress);
110:        _ownerAddress = newOwnerAddress;
111:    }
```

The reason for transferring could be that the owner address is compromised then the protocol would also want to remove the old owner from the `admin` role. This is a separate call however which can be delayed or forgotten. Since the `admin` role has a lot of powers in the system it is crucial it is not given to untrusted parties.

#### Recommendation
Consider to remove the old owner from the `admin` role when `transferOwnership` is called.


### L-12 `totalBattles` will be double what it should be

[`RankedBattle::updateBattleRecord`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L322-L349):
```solidity
File: src/RankedBattle.sol

322:    function updateBattleRecord(
            // ...
328:    ) 
329:        external 
330:    {   
            // ... logic
348:        totalBattles += 1;
349:    }
```

The `totalBattles` field is increased by each call. The issue is that calls are done for each fighter, not each battle. Each battle will have two calls, one for the loser, one for the winner (unless tie).

#### Recommendation
Consider only increasing `totalBattles` when `initiatorBool==true`, as each battle should only have on initiator:
```diff
        if (initiatorBool) {
            _voltageManagerInstance.spendVoltage(fighterOwner, VOLTAGE_COST);
+           totalBattles += 1;
        }
-       totalBattles += 1;
```


### L-13 a GameItems `dailyAllowance` cannot be set larger than `65535`

The parameter in `dailyAllowance` when calling [`GameItems::createGameItem`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L215): is just `uint16`:
```solidity
File src/GameItems.sol

215:        uint16 dailyAllowance
```

However the field in [`GameItems::GameItemAttributes`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L41): is `uint256:`
```solidity
File src/GameItems.sol

41:        uint256 dailyAllowance;
```

#### Recommendation
Consider using `uint256` as type for paramter `dailyAllowance`


### L-14 last wei of `NRN` cannot be minted
When minting `NRN` there is a check to not mint above `MAX_SUPPLY`. Unfortunately it compares using `<` instead of `<=`. Hence the last wei of `NRN` will not be able to be minted:

[`Neuron::mint`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L155-L159):
```solidity
File: src/Neuron.sol

155:    function mint(address to, uint256 amount) public virtual {
            // @audit comparison is `<` hence last wei of NRN cannot be minted
156:        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
157:        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
158:        _mint(to, amount);
159:    }
```

#### PoC
Test in `test/Neuron.t.sol`:
```solidity
    function testMintWithMinterUpToMaxSupply() public {     
        address minter = vm.addr(3);
        _neuronContract.addMinter(minter);

        // NRN left to mint
        uint256 left = _neuronContract.MAX_SUPPLY() - _neuronContract.totalSupply();
        
        vm.startPrank(minter);

        // cannot mint as the last wei cannot be mintd
        vm.expectRevert("Trying to mint more than the max supply");
        _neuronContract.mint(minter, left);

        // minting one wei less works
        _neuronContract.mint(minter, left-1);

        vm.stopPrank();
    }
```

#### Recommendation
Consider comparing to `<= MAX_MINT`
```diff
-       require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
+       require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply");
```

### L-15 win/loss combination are not atomic
When a battle has happened the results are published on-chain:
[`RankedBattle::updateBattleRecord`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L322-L328):
```solidity
File: src/RankedBattle.sol

    function updateBattleRecord(
        uint256 tokenId, 
        uint256 mergingPortion,
        uint8 battleResult,
        uint256 eloFactor,
        bool initiatorBool
    ) 
```

For each fighter there is a call made to `updateBattleRecord`. The issue is that these calls are not done atomically. If the second one fails (say the fighter didn't have enough voltage) the first fighters results are already registered. Since state cannot be rolled back once included on-chain the results of the battle will never be complete.

#### Recommendation
Consider sending both the win and the loss in the same transaction. I.e. change `updateBattleRecord` so that it accepts an array for `tokenIds` and whatever other params that can differ between fighters.


### L-16 fixed amounts can lead to future economic issues
When you create a `GameItem` it is created with [an `itemPrice`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L220-L229). This price cannot be changed once the item is created.

The same goes for the cost of doing a [`reRoll`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L38-L39) of a fighter:
```solidity
File: src/FighterFarm.sol

38:    /// @notice The cost ($NRN) to reroll a fighter.
39:    uint256 public rerollCost = 1000 * 10**18;    
```

Imagine that `AiArena` becomes very popular and lots of people play it. Naturally the price of `NRN` will rise. That will make re-rolls and game items less affordable for normal users, giving rich players an unfair advantage. As they would be able to re-roll and buy batteries.

The price of certain functions in the game are also important balancing points. Hence having the ability to tweak them can enhance the balance of the game in general.

#### Recommendation
Consider adding a way for `admin`/`owner` to change the `itemPrice` of a `GameItem` and also the cost of doing a reRoll.


## Non-critical/Informational


### NC-01 some fields are not included in `FighterFarm::getAllFighterInfo`

The name `getAllFighterInfo` hints that all the info about a fighter should be returned:
[`FighterOps::`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79-L104)
```solidity
File: src/FighterOps.sol

 79:    function viewFighterInfo(
 80:        Fighter storage self, 
 81:        address owner
 82:    ) 
 83:        public 
 84:        view 
 85:        returns (
...
 93:        )
 94:    {
 95:        return (
 96:            owner,
 97:            getFighterAttributes(self),
 98:            self.weight,
 99:            self.element,
100:            self.modelHash,
101:            self.modelType,
102:            self.generation
103:        );
104:    }
```

Comparing to the `Fighter` declaration:
[`FighterOps::Fighter`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L36-L46):
```solidity
File: src/FighterOps.sol

36:    struct Fighter {
37:        uint256 weight;
38:        uint256 element;
39:        FighterPhysicalAttributes physicalAttributes;
40:        uint256 id;
41:        string modelHash;
42:        string modelType;
43:        uint8 generation;
44:        uint8 iconsType;
45:        bool dendroidBool;
46:    }
```

You can see that some fields are left out, `iconsType` and `dendroidBool`.

Consider including these fields in `getAllFighterInfo`


### NC-02 `FighterFarm::mintFromMergingPool` uses `msg.sender` which always is `MergingPool`
When doing the pseudorandom generation of dna, the input uses `msg.sender`:
[`FighterFarm::mintFromMergingPool`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L321-L324):
```solidity
File: src/FighterFarm.sol

321:        require(msg.sender == _mergingPoolAddress);
322:        _createNewFighter(
323:            to, 
                // `msg.sender` will always be `_mergingPoolAddress`
324:            uint256(keccak256(abi.encode(msg.sender, fighters.length))), 
```

This seems like a mistake as in both [`claimFighters`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L214) and [`reRoll`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L379) `msg.sender` is also used but here it means the account doing the call.

Consider using `to` as that is the initiator of the call.


### NC-03 unnecessary voltage check in `RankedBattle`

[`RankedBattle::updateBattleRecord`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L334-L338):
```solidity
File: src/RankedBattle.sol

334:        require(
335:            !initiatorBool ||
336:            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
337:            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
338:        );
```

This check is unecessary as it is already checked in [`VoltageManager::spendVoltage`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L107-L110) which also replenished voltage:
```solidity
File: src/VoltageManager.sol

107:        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
108:            _replenishVoltage(spender);
109:        }
110:        ownerVoltage[spender] -= voltageSpent;
```

Consider removing the check in `RankedBattle::updateBattleRecord`.


### NC-04 Lack of events emitted
Lack of events emitted for important changes can complicate off-chain monitoring. Consider emitting events for the below actions:

#### - When winners are picked in `MergingPool`
A contract admin calls [`MergingPool::pickWinners`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L118-L132) to decide which owners will be able to claim a merged fighter. There is however no event emitted with the winners.

#### - New round in `RankedBattle`
When a new round is started in `RankedBattle`, neither [`RankedBattle::setNewRound`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L233-L239) or [`StakeAtRisk::setNewRound`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L78-L84) emit any events.

#### - Battle results in `RankedBattle`
When a battle is performed, events are only emitted when points are changed or stake put at risk. However, nowhere in [`RankedBattle::updateBattleRecord`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L322-L349) is any event emitted for the results of the fight.

#### - Game item added in `GameItems`
When a new GameItem is added in [`GameItems::createGameItem`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L208-L235) an event is only emitted if the item is locked (non-transferrable). 


### NC-05 `generation` is `uint8` in field but `uint16` in return
When `generation` is defined in [`FighterOps::Fighter`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L43) it is `uint8`:
```solidity
File: src/FighterOps.sol

43:        uint8 generation;
```

But in the return from [`FighterOps::viewFighterInfo`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L92) it is `uint16`.

Consider changing the returned value to be `uint8`


### NC-06 Indentation is off

#### - `RankedBattle::_getStakingFactor`

[`RankedBattle::_getStakingFactor`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L526-L534):
```solidity
File: src/RankedBattle.sol

526:    {
          // @audit only two spaces indentation
527:      uint256 stakingFactor_ = FixedPointMathLib.sqrt(
528:          (amountStaked[tokenId] + stakeAtRisk) / 10**18
529:      );
530:      if (stakingFactor_ == 0) {
531:        stakingFactor_ = 1;
532:      }
533:      return stakingFactor_;
534:    }    
```

#### - `FighterFarm::_ableToTransfer`
[`FighterFarm::_ableToTransfer`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L539-L545):
```solidity
File: src/FighterFarm.sol

539:    function _ableToTransfer(uint256 tokenId, address to) private view returns(bool) {
540:        return (
              // @audit only two spaces indentation
541:          _isApprovedOrOwner(msg.sender, tokenId) &&
542:          balanceOf(to) < MAX_FIGHTERS_ALLOWED &&
543:          !fighterStaked[tokenId]
544:        );
545:    }
```

The contracts in general are indented with 4 spaces. The first level of `RankedBattle::_getStakingFactor` and `FighterFarm::_ableToTransfer` is however only indented with 2 spaces.

Consider indenting the whole `RankedBattle::_getStakingFactor` and `FighterFarm::_ableToTransfer` with 4 spaces as well.


### NC-07 loop variable `currentRound` is confusing
when claiming rewards in RankedBattle there's a loop iterating over all rounds, [`RankedBattle::claimNRN`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L299-L305):
```solidity
File: src/RankedBattle.sol

299:        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
300:            nrnDistribution = getNrnDistribution(currentRound);
301:            claimableNRN += (
302:                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
303:            ) / totalAccumulatedPoints[currentRound];
304:            numRoundsClaimed[msg.sender] += 1;
305:        }
``` 

The loop variable name `currentRound` is confusing here as `roundId` _is_ the current round. Hence reading the code that uses the loop variable becomes very confusing.

Consider just using a normal loop variable like `i` or `r` (for round) in the loop. 

The same applies for [`MergingPool::claimRewards`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L149-L163) and [`getUnclaimedRewards`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L176-L183).


### NC-08 usage of sensitive words

[`AiArenaHelper::dnaToIndex`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L176):
```solidity
File: src/AiArenaHelper.sol

176:        uint256 cumProb = 0;
```

`cumProb` has an equivocal meaning where the other interpretation is less than desierable.

Consider using another word, like `sumProb` or `totProb`.


### NC-09 some test assertions overly complicated

using [`RankedBattleTest::testStakeNRN`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/test/RankedBattle.t.sol#L243) as an example:
```solidity
File: test/RankedBattle.t.sol

243:        assertEq(_neuronContract.balanceOf(staker) >= 4_000 * 10 ** 18, true);
```

This is overly complicated and can be rewritten using [`assertGe`](https://book.getfoundry.sh/reference/ds-test#asserting):
```solidity
assertGe(_neuronContract.balanceOf(staker),4_000 * 10 ** 18);
```


### NC-10 misspelled test
[`VoltageManager::testUseVolatgeBattery`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/test/VoltageManager.t.sol#L127):
```solidity
File: test/VoltageManager.t.sol

127:    function testUseVolatgeBattery() public {
```

Consider renaming the test to `testUseVoltageBattery`.