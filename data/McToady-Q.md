## [I-01] Precision loss in RankedBattle NRN distribution
Actual $NRN claimable after a round in `RankedBattle` will likely be less than `rankedNrnDistribution` due to rounding errors.

**PoC**
Log showing this: 
```solidity
  Alice NRN 1206896551724137931034
  Bob NRN 1724137931034482758620
  Chad NRN 2068965517241379310344
  Dave NRN 0
  Error: a == b not satisfied [uint]
        Left: 4999999999999999999998
       Right: 5000000000000000000000
```
Add the following foundry test to `RankedBattle.t.sol` to recreate this:
```solidity
    function testClaimNRNNoPrecisionLoss() public {
        address alice = makeAddr("alice");
        address bob = makeAddr("bob");
        address chad = makeAddr("chad");
        address dave = makeAddr("dave");
        
        address[] memory users = new address[](4);
        users[0] = alice;
        users[1] = bob;
        users[2] = chad;
        users[3] = dave;
        
        // Set up all 4 users with a fighter, and have them stake 4k $NRN
        for (uint256 i; i < users.length; ++i) {
            address user = users[i];
            _mintFromMergingPool(user);
            _fundUserWith4kNeuronByTreasury(user);
            _fundUserWith4kNeuronByTreasury(user);
            uint256 stakeAmount = (i + 1) * 200 ether;
            vm.prank(user);
            _rankedBattleContract.stakeNRN(stakeAmount, i);
        }

        // Alice beats bob
        _rankedBattleContract.updateBattleRecord(0, 50, 0, 1500, false);
        _rankedBattleContract.updateBattleRecord(1, 50, 2, 1500, true);
        // Chad beats dave
        _rankedBattleContract.updateBattleRecord(2, 50, 0, 1500, false);
        _rankedBattleContract.updateBattleRecord(3, 50, 2, 1500, true);
        // Bob beats dave twice
        _rankedBattleContract.updateBattleRecord(1, 50, 0, 1500, false);
        _rankedBattleContract.updateBattleRecord(3, 50, 2, 1500, true);
        _rankedBattleContract.updateBattleRecord(1, 50, 0, 1500, false);
        _rankedBattleContract.updateBattleRecord(3, 50, 2, 1500, true);

        for (uint256 i; i < users.length; ++i) {
            uint256 userPoints = _rankedBattleContract.accumulatedPointsPerAddress(users[i], 0);
            console.log("User Points", userPoints);
        }
        vm.stopPrank();

        _rankedBattleContract.setNewRound();

        uint256 aliceNRN = _rankedBattleContract.getUnclaimedNRN(alice);
        uint256 bobNRN = _rankedBattleContract.getUnclaimedNRN(bob);
        uint256 chadNRN = _rankedBattleContract.getUnclaimedNRN(chad);
        uint256 daveNRN = _rankedBattleContract.getUnclaimedNRN(dave);

        console.log("Alice NRN", aliceNRN);
        console.log("Bob NRN", bobNRN);
        console.log("Chad NRN", chadNRN);
        console.log("Dave NRN", daveNRN);
        assertEq((aliceNRN + bobNRN + chadNRN + daveNRN), _rankedBattleContract.rankedNrnDistribution(0));
    }
```

On top of this, in a real scenario with more games, more players competing, more varying staking amounts and differing `eloFactors` the rounding errors would become even more apparent.

## [I-02] Wallet limit in FighterFarm could cause issues for AAMintPass holders with more than 10 passes

The FighterFarm contract ensures that a wallet cannot own more than `MAX_FIGHTERS_ALLOWED` (10), whereas there is no similar check on number of `AAMintPass` tokens a user can hold. Therefore if a user with more than 10 mint passes would have to transfer their additional tokens to another address in order to successfully call `redeemMintPasses`

## [I-03] If a user calls `FighterFarm::redeemMintPass` and `mintpassIdsToBurn > 10` the function will waste a lot of gas before eventually reverting

The FighterFarm contract ensures that one wallet can only own `MAX_FIGHTERS_ALLOWED` (10) number of tokens. During `redeemMintPass` this is checked in each loop iteration during `_createNewFighter` with the following line:
`require(balanceOf(to) > MAX_FIGHTERS_ALLOWED);`
If for example the user attempts to mint 11 tokens, the gas will be spent creating the first 10 tokens before reverting on the 11th.

**Mitigation**
It would be beneficial at the start of `redeemMintPass` to add a check such as:
`require(balanceOf(msg.sender) + mintPassIdsToBurn.length <= MAX_FIGHTERS_ALLOWED, "Exceeded max fighters");`
This way a users transaction will revert early and save them wasting a large amount of gas.

## [I-04] If length of arrays passed to MergingPool::claimRewards are less than the users number of wins the function will revert with an out of bounds array access error

There is no option in the contract to claim singular rewards and the `claimRewards` function loops through all unclaim rounds to mint any outstanding rewarsd the user has. As well as this it's not possible for the funciton call to know how many win's the user is going to have so can't validate array lengths early to avoid wasting gas.
Therefore it's important for the user to know how many NFTs they can claim before calling `claimRewards` to avoid reverting at the out of bounds array access error after using gas to mint the intial tokens. The project should be aware of this and tries to lead users towards the `getUnclaimedRewards` function before they call `claimRewards`.

## [I-05] `MergingPool::pickWinner` should be renamed `setWinner` to be clearer about it's functionality

The function `pickWinner` does not actually pick the winner in the MergingPool contract as that is already done offchain, as shown by the `winners` array that is passed as an argument. 
Changing this would improve the code's readability and avoid any confusion from users who may expect the winner is selected onchain rather than being picked offchain and then having the result stored on chain afterwards.

## [I-06] Unclear constant token amounts in Neuron.sol

See the following example:
```solidity
    /// @notice Initial supply of NRN tokens to be minted to the treasury.
    uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;

    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
    uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;

    /// @notice Maximum supply of NRN tokens.
    uint256 public constant MAX_SUPPLY = 10**18 * 10**9;

```

It's difficult to work out what these values actually are and what percentage of `MAX_SUPPLY` the other two values represent. It would be improve the code's readability to use either the keyword `ether` or `1e18` to make it clearer to the reader what these values are.

Example:
```solidity
    /// @notice Initial supply of NRN tokens to be minted to the treasury.
    uint256 public constant INITIAL_TREASURY_MINT = 200_000_00 ether;

    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
    uint256 public constant INITIAL_CONTRIBUTOR_MINT = 500_000_000 ether;

    /// @notice Maximum supply of NRN tokens.
    uint256 public constant MAX_SUPPLY = 100_000_000_000 ether;

```

## [I-07] Neuron::claim allowing partial claims is unnecessary

It's unclear why a user would only want to claim only part of their allotted airdrop, so it would be better remove the `amount` argument and just send the user all the tokens they're entitled to.

**Recommendation**
Example here:
```solidity
    function claim() external {
        uint256 amountToClaim = allowance(treasuryAddress, msg.sender);
        require(amountToClaim > 0, "Nothing to claim");

        transferFrom(treasuryAddress, msg.sender, amountToClaim);
        emit TokensClaimed(msg.sender, amountToClaim);
    }
```

## [I-08] Error messages in Neuron.sol's require blocks suggest the error comes from the ERC20 code when this is not the case

It's typical of frequently inherited contracts to include the contract name in their error messages to make it clearer for users to understand the issue and for developers to debug during testing. However the Neuron contract has multiple errors messages labelled "ERC20" which can make it confusing to understand where the origin of the error actually is.
It's recommended to remove the "ERC20" tag to make this clearer.

## [I-09] No way to revoke `MINTER_ROLE`, `STAKER_ROLE` or `SPENDER_ROLE` in Neuron.sol should it be necessary.

Once an address if given the permission to mint Neuron tokens or spend tokens on behalf of users there is no way to revoke these permissions should anything go wrong. Given the amount of power these functions have over the $NRN token, any mistake where these are set incorrectly or one of the addresses is compromised would cause irrepairable damage to the protocol.

**Recommendation**
Change the `addX(address newXAddress)` functions to `setX(address XAddress, bool permitted)`. This would give the protocol time to act should something happen to lessen the possible damage done.

## [I-10] In `RankedBattle::_neuronInstance` should only be allowed to be set once, as changing the token address once the contract is in use would break the system
The function `RankedBattle::instantiateNeuronContract` can be called by `_ownerAddress` with no checks that the function has not already been called. This should be changed to preferably set the `_neuronInstance` in the constructor or via a function that can only be called once. Any change to the contract address once users have staked the original address' tokens would break the contracts accounting as users would have `stakedAmount`of the old contract's tokens.

## [I-11] `RankedBattle` unused library implementation
The `RankedBattle` contract contains the following line: `using FixedPointMathLib for uint`.
However the one use of the library declares the library again `_getStakingFactor` function here:
`uint256 stakingFactor_ = FixedPointMathLib.sqrt((amountStaked[tokenId] + stakeAtRisk) / 10**18);`
**Recommendation**
`using FixedPointMathLib for uint` can be removed from the code to improve clarity without causing any issues to the codebase.

## [I-12] Users can call `FighterFarm::reRoll` with an incorrect `fighterType`

**Description**
If a user holds an NFT where `fighterType == 0` they can still call `reRoll(tokenId, 1)` and there is no check that they are actually a dendroid.
If this is called the rerolled fighter will always have the value `1` for each attribute because the following line in `_createFighterBase` sets the `dna` value to 1 throwing off the pseudorandom trait generation:
`uint256 newDna = fighterType == 0 ? dna : uint256(fighterType);`

**Proof Of Concept**
Add the following test to `FighterFarm.t.sol` to confirm this:
```solidity
  function testTryRerollFighterIntoDendroid() public {
        // Claim one non dendroid fighter
        uint8[2] memory numToMint = [1, 0];
        bytes memory claimSignature = abi.encodePacked(
            hex"407c44926b6805cf9755a88022102a9cb21cde80a210bc3ad1db2880f6ea16fa4e1363e7817d5d87e4e64ba29d59aedfb64524620e2180f41ff82ca9edf942d01c"
        );
        string[] memory claimModelHashes = new string[](1);
        claimModelHashes[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        string[] memory claimModelTypes = new string[](1);
        claimModelTypes[0] = "original";

        _fighterFarmContract.claimFighters(numToMint, claimSignature, claimModelHashes, claimModelTypes);       

        // Give contract some $NRN to pay reroll fee
        _fundUserWith4kNeuronByTreasury(address(this));
        // Give fighter farm spender role to transfer $NRN
        _neuronContract.addSpender(address(_fighterFarmContract));

        // Call reroll with fighterType = 1
        _fighterFarmContract.reRoll(0, 1);

        // Confirm the attributes are now all 1
        ( , uint256[6] memory attributes, , , , ,) = _fighterFarmContract.getAllFighterInfo(0);
        assertEq(attributes[0], 1);
        assertEq(attributes[1], 1);
        assertEq(attributes[2], 1);
        assertEq(attributes[3], 1);
        assertEq(attributes[4], 1);
        assertEq(attributes[5], 1);

    }
```

**Recommended Mitigation**
Remove the `fighterType` argument from `FighterFarm::reRoll` and instead call `fighters[tokenId].dendroidBool` to figure out what the fighters type is.

## [I-13] Fighter that does not intiate battle in `RankedBattle` still loses points, despite what is stated in the projects documentation

The following lines are in the [projects documentation](https://docs.aiarena.io/gaming-competition/nrn-rewards):
```
"To keep terms consistent, we will use “Challenger” and “Opponent” to denote the NFTs instigating and being matched for Ranked Battles, respectively.  

It is important to note that only the Challenger NFT accrues Points during a match. For the Opponent NFT, the outcome does not impact their Point total. However, it does impact their Elo score¹."
```

However it is clear from the logic within `RankedBattle::updateBattleRecord` that `initiatorBool` is only used to calculate if a fighter should be deducted voltage for the battle or not.

**Recommendation**
The project should update their documentation or rewrite this function to match what is stated in the docs.

## [I-14] `RankedBattle::totalBattles` variable gets incremented twice per battle
The `totalBattles` variable will end up being twice as many battles as it should because it is incremented in `updateBattleRecord` which will be called twice per battle (once for the winner, once for the loser)

**Recommendation**
Only increment `totalBattles` if `initiatorBool` is true. This will keep it to only being updated once per battle. 

## [I-15] `msg.sender` used `FighterFarm::mintFromMergingPool`'s dna generation will always be `_mergingPoolAddress`

The pseudorandom hash used by `mintFromMergingPool` to create a fighters `dna` is actually deterministic and easily predictable.
See the following line:
`uint256(keccak256(abi.encode(msg.sender, fighters.length))),`
As `msg.sender` will always be `_mergingPoolAddress` and `fighters.length` will increment from 0 upwards it would be possible for a user to calculate the `dna` of future fighters in advance and wait until `fighters.length` is a value that creates their desired traits before calling `MergingPool::claimRewards`.

**Mitigation**
The project should either use Chainlink VRF to provide a non-deterministic source of random number generation, or at the very least add other variables to the has such as `block.timestamp` that will at least make fishing for a specific value less simple.

## [I-16] No zero quantity check on `GameItems::mint` can lead to users wasting gas
If a user calls `mint` with a `quantity` of zero the function call will not revert, emitting multiple unnecessary events and wasting the users gas.

## [I-17] `RankedBattle::globalStakedAmount` is incorrect due to amounts lost to `stakeAtRisk` contract
The variable `globalStakedAmount` is supposed to track the overall staked amount in the system, however when stake is sent to the StakeAtRisk contract (or when at risk stake is swept to the `_treasuryAddress` the total `globalStakedAmount` is not decreased. 
This means the variable gives an unclear indication of actual amount staked within the RankedBattle contract.

## [I-18] Multiple `_neuronInstance.transfer` calls that do not revert if unsuccessful
Affected:
`RankedBattle::_addResultPoints`
`RankedBattle::stakeNRN`
`RankedBattle::unstakeNRN`
`GameItems::mint`
`FighterFarm::reRoll`
`StakeAtRisk::reclaimNRN`

The following logic is present in all the above functions:
```solidity
        // State updates

        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
        if (success) {
            // More state updates
        }
```

This logic ensures that only the second group of state updates are made if the transfer is unsuccessful, however the state updates made before the call remain valid when they should not be. One example of this is in `RankedBattle::unstakeNRN` where `amountStaked[tokenId] -= amount` is changed before the Neuron transfer, meaning that if the transfer returns false, the users `amountStaked` would be decreased but they would not receive their tokens.

**Recomendation**
It would be better to complete all state changes prior to `_neuronInstance.transfer` and then revert the whole transaction `if (!success)`. 

## `FighterFarm::_ableToTransfer` would be better suited as a modifier
The function `_ableToTransfer` is just responsible for checking a token id is able to be transferred, so it would be better suited as a modifier added to `transferFrom` and `safeTransferFrom`.