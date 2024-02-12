### `transferOwnership` doesn't revoke admin rights from the old owner
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L64
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L85
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L108
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L89
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L167

The owner is granted admin rights when the contract is created
```solidity
    constructor(address ownerAddress, address gameItemsContractAddress) {
        _ownerAddress = ownerAddress;
        _gameItemsContractInstance = GameItems(gameItemsContractAddress);
>>      isAdmin[_ownerAddress] = true;
    } 
```
When transferring ownership to another address, the old owner still retains that role. Consider setting `isAdmin` for the old owner to `false`.

### User can bypass voltage limit by participating in battles from different addresses

Initiator of the ranked battle needs to spend 10 voltage units to participate
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L345-L347
```solidity
    function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }
        ownerVoltage[spender] -= voltageSpent;
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }
```
The voltage is replenished after 24 hours or by sacrificing a special NFT
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L117
```solidity
    function _replenishVoltage(address owner) private {
>>      ownerVoltage[owner] = 100;
        ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days);
    }  
```
As we can see, the voltage is bound to the fighter owner's address, thus if owner has multiple fighters, he can split them between his wallets, increasing the number of matches he can participate in during the day. This includes staked and unstaked matches.

### Signature can be replayed in different chains
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L191-L206
Users are allowed to mint a fighter token by providing a signature from an address owned by the protocol. If Ai-Arena will deploy in other networks in the future, this signature can be replayed.

### `FighterFarm.sol` lacks setter for the `_delegatedAddress`
`_delegatedAddress` is used to sign fighter claim messages and setting token URIs, but unlike other instances such as `NeuronToken` or `MergingPool`, it cannot be changed.

### The probability array is initialized twice
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L45-L51
```solidity
    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
>>      addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
>>      for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```

`addAttributeProbabilities(0, probabilities)` performs the same function as the for loop below.

### No 0 divisor check in `addAttributeDivisor` function
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L74
Since we are using elements `attributeToDnaDivisor` elements as divisors, additional check is needed to validate that these elements !=0
```solidity
uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;
```

### `addAttributeProbabilities` can use more validation
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131
`attributeProbabilities` elements correspond to the likelihood of acquiring a particular physical attribute. If the sum of the elements exceeds 100, some attributes will not be available.
```solidty
        for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
            cumProb += attrProbabilities[i];
            if (cumProb >= rarityRank) { // rarityRank is a number 0-100
                attributeProbabilityIndex = i + 1;
                break;
            }
        }
```

### `viewFighterInfo` can be more verbose
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79
`viewFighterInfo` function doesn't return the fighter type or icon type.

### Default admin is not initialized
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L68-L76
Neuron token inherits from the OZ AccessControl contract, instead of using DEFAULT_ADMIN_ROLE it relies on custom functions like the one below
```solidity
    function addMinter(address newMinterAddress) external {
        require(msg.sender == _ownerAddress);
>>      _setupRole(MINTER_ROLE, newMinterAddress);
    }
```
This makes role management less flexible because there is no way to revoke roles if needed. Consider granting DEFAULT_ADMIN_ROLE to the owner in the constructor. 

### `setupAirdrop` can overwrite unclaimed approvals
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L127-L134
```solidity
    function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
        require(isAdmin[msg.sender]);
        require(recipients.length == amounts.length);
        uint256 recipientsLength = recipients.length;
        for (uint32 i = 0; i < recipientsLength; i++) {
>>          _approve(treasuryAddress, recipients[i], amounts[i]);
        }
    }
```
If the airdrop consists of multiple stages, users who do not claim their tokens using `claim` function risk having their approval overwritten in the next stage.
```solidity
    // OZ ERC20 contract
    function _approve(
        address owner,
        address spender,
        uint256 amount
    ) internal virtual {
        require(owner != address(0), "ERC20: approve from the zero address");
        require(spender != address(0), "ERC20: approve to the zero address");

>>      _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }
``` 
Consider using `increaseAllowance` instead.

### `burnFrom` shouldn't decrease allowance in case it's `type(uint256).max`
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L201
If spender has an infinite allowance we can burn tokens without changing `_allowances` value.

### Unusual setter for NRN distribution
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L220
`setRankedNrnDistribution` takes the NRN amount without the 10 ** 18 multiplier, most Dapps use X * 10 ** (decimal) amounts as arguments and it's pretty easy to make a mistake and enter such a value into `setRankedNrnDistribution` too.

### `globalStake` value is not updated if staker loses his stake in battle
`globalStake` variable is used to track total amount of NRN tokens that was staked in the `RankedBattle.sol` contract.
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L257
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L276
Unfortunately it's is not updated when part of the stake that was at risk during a round is transferred to the treasury at the end of the round.
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L491-L498
```solidity
            } else {
                /// If the fighter does not have any points for this round, NRNs become at risk of being lost
                bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
                if (success) {
                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                    amountStaked[tokenId] -= curStakeAtRisk;
                }
            }
        }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L142
```solidity
    /// @notice Sweeps the lost stake to the treasury contract.
    /// @dev This function is called internally to transfer the lost stake to the treasury contract.
    function _sweepLostStake() private returns(bool) {
        return _neuronInstance.transfer(treasuryAddress, totalStakeAtRisk[roundId]);
    }
```

### The user risks receiving an OOG error if he doesn't claim NRN rewards for too long.
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L299-L305
`claimNRN` function iterates through all rounds in which user didn't claim to calculate the final reward, if a substantial number of rounds has passed since the last claim, the function can revert with an out of gas error.
```solidity
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound];
            numRoundsClaimed[msg.sender] += 1;
        }
```
Consider allowing the user to specify the number of rounds for which he would like to be rewarded.
