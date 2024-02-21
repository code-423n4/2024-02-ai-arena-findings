
# QA Report

## Summary

| id | title |
| --- | --- |
| [L-1](#L-1-reclaim-issues-in-updateBattleRecord) | reclaim issues in updateBattleRecord |
| [L-2](#l-2-neuron.setupAirdrop-is-succeptable-to-approval-race-condition) | neuron.setupAirdrop is succeptable to approval race condition |
| [L-3](#l-3-Signature-verification-is-not-EIP-712-complaint) | Signature verification is not EIP-712 complaint|
| [L-4](#l-4-if-all-stakes-were-slashed-then-the-fighter-owner-can-experience-DOS-to-unlock-his-staked-fighter) | if all stakes were slashed then the fighter owner can experience DOS to ulock his staked fighter |
| [L-5](#L-5-Wrong-FixedPointMathLib-version-usage) | Wrong FixedPointMathLib version usage|
| [L-6](#L-6-FighterFarm::reroll-doesn't-update-the-generation-and-other-item-data) | FighterFarm::reroll doesn't update the generation and other item data |
| [L-7](#l-7-neuron.burnFrom-is-not-in-erc-standard) | neuron.burnFrom is not in erc standard |
| [L-8](#l-8-MergingPool.getFighterPoints-will-revert-if-`id->-1`-as-input) | MergingPool.getFighterPoints will revert if `id > 1` as input|
| [L-9](#l-9-wrong-support-interface-chain-inheritance) | wrong support interface chain inheritance |
| [L-10](#L-10-precision-loss-an-staking-factor-calculation) | precision loss an staking factor calculation|
| [L-11](#L-11-Transferring-FighterFarm-tokens-to-a-address-with-9-holders-will-DOS-their-purchases) | Transferring FighterFarm tokens to a address with 9 holders will DOS their purchases |
| [L-12](#L-12-FighterFarm::mintFromMergingPool-uses-msg.sender-instead-of-to-address-to-compute-dna) | FighterFarm::mintFromMergingPool uses msg.sender instead of to  address to compute dna |
| [L-13](#l-13-FighterFarm::updateModel-allows-an-one-to-copy-the-model-of-others) |  FighterFarm::updateModel allows an one to copy the model of others |
| [L-14](#l-14-Reroll-cost-is-hardcoded) |  Reroll cost is hardcoded |
| [L-15](#l-15-claimNRN-can-be-DOS-if-rains-go-over-100+) | claimNRN can be DOS if rains go over 100+ |
| [L-16](#l-16-Support-for-the-PUSH0-opcode) | Support for the PUSH0 opcode |


## Low




### L-1 reclaim issues in updateBattleRecord 

1.  updateBattleRecord will revert  if stake at risk doesnt have enough balance.
 if stakeAtRisk contract doesn't have any balance, then the `updateBattleRecord` by game server will revert. This balance descepency will happen if some winners reclaim larger than others or many times.

2. the reclaim % is very unfair, because he stake at risk is 1% of the staked when the attle is lost, but when won next time we only get 1% of that 1%, meaning 10000 tokens initially staked will risk 10 tokens as loss. now any win after that will only be possible to reclaim  % of the 10 tokens = 0.1 token. This is unfair, and it takes 100 wins to reclaim all that was lost of 1%. So design a better system, of reclaiming %

```solidity
    function _addResultPoints(
        uint8 battleResult, 
        uint256 tokenId, 
        uint256 eloFactor, 
        uint256 mergingPortion,
        address fighterOwner
    ) 
        private 
    {
        ------------------------------- skimmed --------------------------
            if (curStakeAtRisk > 0) {
                _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner); 
                amountStaked[tokenId] += curStakeAtRisk;
            }   
        ------------------------------- skimmed --------------------------
    }

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
            amountLost[fighterOwner] -= nrnToReclaim;
            emit ReclaimedStake(fighterId, nrnToReclaim);
        }
    }

```


### L-2 neuron.setupAirdrop is succeptable to approval race condition

if admin wants to edit already set airdrop then he will change from 5 to 10, now the recipient will frontrun the second setup and claim 5, then after the second set he will claim 15 making total claim = 15. So make setup to zero first and then set up to value like USDT mainnet., add a line like if current approval is non-zero then make it zero and then only you can make new non zero.


```solidity
    function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
        require(isAdmin[msg.sender]);
        require(recipients.length == amounts.length);
        uint256 recipientsLength = recipients.length;
        for (uint32 i = 0; i < recipientsLength; i++) {
            _approve(treasuryAddress, recipients[i], amounts[i]);
        }
    }
```

### L-3 Signature verification is not EIP-712 complaint

signature doesn't encode contract address, version, chainId, and other replayablility. So it is recommended to use Openzeppelin ECDSA library implemntation to verify signatures instead of Verification.sol implementation.

The important check is the below issue

```solidity
   // If your library generates malleable signatures, such as s-values in the upper range, calculate a new s-value
    // with 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141 - s1 and flip v from 27 to 28 or
    // vice versa. If your library also generates signatures with 0/1 for v instead 27/28, add 27 to v to accept
    // these malleable signatures as well.
```
Replace [Verification.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Verification.sol#L6)  with ECDSA.sol from ==> https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d/contracts/utils/cryptography/ECDSA.sol



### L-4 if all stakes were slashed then the fighter owner can experience DOS to unlock his staked fighter

Look at actions [RankedBattle::updateBattleRecord](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L322) and [RankedBattle::_addResultPoints](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L416-L500)

Lets say the loss bps is 50 at some point and nor within 20 battles loss, all the staked amount of a fighter is slashed at put to risk on stakeAtRisk contract. And now the fighter can never call unstake to unlock his locked nft. And its a DOS. so, user has to stake a small amount again and unstake to release the nft.

It is recommended to addunstake nft action if staked amount  == 0 on  [RankedBattle::_addResultPoints](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L416-L500)



### L-5 Wrong FixedPointMathLib version usage

The squareRoot function from `FixedPointMathLib` is much different than the solmate latest version, making the usage of old version vulnerable. so use the latest version at ===> https://github.com/transmissions11/solmate/blob/main/src/utils/FixedPointMathLib.sol




### L-6 FighterFarm::reroll doesn't update the generation and other item data

when reolling to new skin/weight, the game item data is nt fulling updated.

```diff
function reRoll(uint8 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]); 
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

        _neuronInstance.approveSpender(msg.sender, rerollCost);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
        if (success) {
            numRerolls[tokenId] += 1;
            uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
            (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
            fighters[tokenId].element = element;
            fighters[tokenId].weight = weight;
+           fighters[tokenId].generation = generation[fighterType];
+           fighters[tokenId].dendroidBool = fighterType == 1;
            fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
                newDna,
                generation[fighterType],
                fighters[tokenId].iconsType,
                fighters[tokenId].dendroidBool
            );
            _tokenURIs[tokenId] = "";
        }
    } 
```   




### L-7 neuron.burnFrom is not in erc standard

- If the approval is type(uint).max, then approval should not be decreased thats the standard. Bur neuron.burn decreases. And it supposed to inherit erc20 burnable externsion, instead of manuall modifying the burn.

```solidity
    function burnFrom(address account, uint256 amount) public virtual {
        require(
            allowance(account, msg.sender) >= amount, 
            "ERC20: burn amount exceeds allowance"
        );
        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
        _burn(account, amount);
        _approve(account, msg.sender, decreasedAllowance);
    }
```





### L-8 MergingPool.getFighterPoints will revert if `id > 1` as input

- look at the difference to understand the issue. Because, if id > 1 then loop will  revert. because it loop from 0 to that id but length is hardcoded to 1.


```diff
    function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
-       uint256[] memory points = new uint256[](1);
+       uint256[] memory points = new uint256[](maxId);
        for (uint256 i = 0; i < maxId; i++) {
            points[i] = fighterPoints[i]; 
        }
        return points;
    }
```


### L-9 wrong support interface chain inheritance

Modify like this, FighterFarm::supportsInterface

```diff
    function supportsInterface(bytes4 _interfaceId)
        public
        view
        override(ERC721, ERC721Enumerable)
        returns (bool)
    {
-       return super.supportsInterface(_interfaceId);
+       return ERC721.supportsInterface(_interfaceId) ||ERC721Enumerable.supportsInterface(_interfaceId);  
    }
```


### L-10 precision loss an staking factor calculation

- diving by 10**18 makes the input for `sqrt` very low, and the result will mostly be in the low precision range.
- So dont divide by 10**18.

```solidity
    function _getStakingFactor( uint256 tokenId,   uint256 stakeAtRisk)  private  view returns (uint256) {

      uint256 stakingFactor_ = FixedPointMathLib.sqrt( (amountStaked[tokenId] + stakeAtRisk) / 10**18 );
      if (stakingFactor_ == 0) stakingFactor_ = 1;
      
      return stakingFactor_;
    }    
```


### L-11 Transferring FighterFarm tokens to a address with 9 holders will DOS their purchases

This is a DOS for claim / mint / redeem / marketplace purchases
- If an address with 9 fighter want to buy / mint any token, then their balalnce can be increased by someone donating a token, reaching `MAX_FIGHTERS_ALLOWED`. This is temporary DOS and he has to butn/trnasfe to other address inorder to claim new nft. 
- If possible, allow token holders have an option to accept tokens only from whitelisted addresses.


### L-12 FighterFarm::mintFromMergingPool uses msg.sender instead of to  address to compute dna

- The redeem and claim functions rely on `to` address for dna computation ex: [here](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L214)

- but here it is called by `_mergingPoolAddress`, so it always uses msg.sender insteaf of `to`

```diff
    function mintFromMergingPool(
        address to, 
        string calldata modelHash, 
        string calldata modelType, 
        uint256[2] calldata customAttributes
    ) 
        public 
    {
        require(msg.sender == _mergingPoolAddress);
        _createNewFighter(
            to, 
-           uint256(keccak256(abi.encode(msg.sender, fighters.length))), 
+           uint256(keccak256(abi.encode(to, fighters.length))), 
            modelHash, 
            modelType,
            0,
            0,
            customAttributes
        );
    }
```




### L-13 FighterFarm::updateModel allows an one to copy the model of others

And no event is emitted.

```diff
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

+       emit UpdateModel(tokenId, modelHash, modelType);
    }
```

### L-14 Reroll cost is hardcoded

    `uint256 public rerollCost = 1000 * 10**18;`

- No way to modify this, at some point no one will reroll just because the price of tokens are too much. So implement a function to modify cost
- Also allow to change the price of the game item, it is also hardcoded.



### L-15 claimNRN can be DOS if rains go over 100+

Isuue is also present in `MergingPool::claimRewards` when unbounded loop will revert OOG error

This issue is due to the unbounded array out of gas error
If some early member came back after 100+ rounds he has to loop all 100 and it will take 100 storage reads + 100 storage updates and will run out of gas. so it is recommended to either update `numRounds` state outside the loop or allow to claim by mentioning particular round by claimer.


```solidity
function claimNRN() external {
        require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
        uint256 claimableNRN = 0;
        uint256 nrnDistribution;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound];
@--->       numRoundsClaimed[msg.sender] += 1;
        }
        if (claimableNRN > 0) {
            amountClaimed[msg.sender] += claimableNRN;
            _neuronInstance.mint(msg.sender, claimableNRN);
            emit Claimed(msg.sender, claimableNRN);
        }
    }
```


### L-16  Support for the PUSH0 opcode

On Arbitrum and Base the PUSH0 opcode is not supported yet. Since the project is using a solidity
version is floating it can only be used with evm version lower than the default shanghai,
e.g. paris.
Make sure to check the support for the aforementioned opcode on all the chains you are planning to
deploy the contracts on


Recommendation
One of the options is to add the following to the foundry.toml file:
1 evm_version = "paris" 

