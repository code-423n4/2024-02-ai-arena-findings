## [L-01] Voltage replenishment will fail in 81.98 years

In roughly 81.98 years, assuming the AI Arena contract still functions, rather unlikely; voltage replenishment will fail for fighters in the Arena because at that point the `block.timestamp` will be too much of a value the `uint32` type can accomodate, hence leading transactions to the `spendVoltage` >> `_replenishVoltage` functions to fail.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L119

```js
/// @notice Replenishes voltage and sets the replenish time to 1 day from now
    /// @dev This function is called internally to replenish the voltage for the owner.
    /// @param owner The address of the owner
    function _replenishVoltage(address owner) private {
        ownerVoltage[owner] = 100;
    @>  ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days); // @audit-info will fail in 81.98 years
    }
```

Switching to a bigger datatype such as the `uint64` or more should be fine for a long time:

```diff
- ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days);
+ ownerVoltageReplenishTime[owner] = uint64(block.timestamp + 1 days);
```

## [L-02] Players can burn 1 full battery even at 90% voltage

The function is called when when a voltage battery to replenish voltage for a player in AI Arena. But it allows players to burn his battery even at 90% voltage which will also lose them the `VoltageReplenishTime`.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L93

```solidity
    function useVoltageBattery() public {
        require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }

```

## [L-03] Players can only claim one of two signatures given by the server using `claimFighters`

The issue here is that if players get first signature and he doesn't claim and he will get the second one. He can't use both now because `nftclaimed` mapping is changed after first claim. And server uses previous `nftClaimed` mapping state for signature generation.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L199C1-L205C13

```solidity
    bytes32 msgHash = bytes32(keccak256(abi.encode(
            msg.sender,
            numToMint[0],
            numToMint[1],
            nftsClaimed[msg.sender][0],
            nftsClaimed[msg.sender][1]
        )));
```

## [L-04] Use Oppenzeppelin's `erecover` function to avoid signature replay due to duplicate v value

The built-in EVM precompile ecrecover is susceptible to signature malleability (because of non-unique s and v values) which could lead to replay attacks. We recommend to use Oppenzeppelin's `erecover` function to avoid signature replay due to duplicate v value.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L206

```solidity
require(Verification.verify(msgHash, signature, _delegatedAddress)); // @audit better to use hashstruct for signing data for better readability and security
```

## [L-05] Transaction does not revert if the `transferFrom` fails and can lead to unexpected impacts

There are many instances of this issue with different impacts.

1. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L376

This will lead to unused approval.
```solidity
function reRoll(uint8 tokenId, uint8 fighterType) public { //@note can you give this any fighter type that you want ?
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

        _neuronInstance.approveSpender(msg.sender, rerollCost); // @note approves this contract
@>      bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost); // @audit should use safetransferFrom

```

2. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L270

In this case, if call fails, it can actually lock the user's funds in the contract since the `amountStaked` is updated before. Plus the user will not be able to transfer or sell his fighter NFT until next round.

```solidity
    function unstakeNRN(uint256 amount, uint256 tokenId) external {
        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
        if (amount > amountStaked[tokenId]) {
            amount = amountStaked[tokenId];
        }
        amountStaked[tokenId] -= amount;
        globalStakedAmount -= amount;
        stakingFactor[tokenId] = _getStakingFactor(
            tokenId,
            _stakeAtRiskInstance.getStakeAtRisk(tokenId)
        );
        _calculatedStakingFactor[tokenId][roundId] = true;
        hasUnstaked[tokenId][roundId] = true;
@>        bool success = _neuronInstance.transfer(msg.sender, amount);
        if (success) {
            if (amountStaked[tokenId] == 0) {
                _fighterFarmInstance.updateFighterStaking(tokenId, false);
            }
            emit Unstaked(msg.sender, amount);
        }
    }
```

3. https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L163

This will lead to unused approval.

```solidity
  _neuronInstance.approveSpender(msg.sender, price);
@>        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, price);
```

## [L-06] If `totalSupply()` is reached fast no more NRN tokens can be minted to fuel the Games

It will take approximately 60k rounds to reach 300 million NRN tokens at which point there will be no more tokens left to fuel game rewards. While the current reward being set at 5000 NRN tokens is great, it can be further reduced the more the supply gets diluted so that the number of maximum rounds can go past 60k.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L155

```
    function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```
## [L-07] `fighterCreatedEmitter` can be called by everyone spamming the ` FighterCreated` event

With the current implementation, anyone can call the fighterCreatedEmitter function of the FighterOps contract. Even though the events are not used offchain right now, they could be in the future which would fool off-chain services that a fighter has been created when in fact it wasn't created. The function lacks access control and can be called directly on-chain.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L530

```
function fighterCreatedEmitter(
        uint256 id,
        uint256 weight,
        uint256 element,
        uint8 generation
    ) 
@>      public // @audit accessible to anyone
    {
        emit FighterCreated(id, weight, element, generation);
    }
```

## [L-08] There is no constraint on the `attributeProbabilities` mapping to have a sum of 100

The probabilities sum is used for the calculation in the function `dnaToIndex`. It doesn't make sense for this mapping to not have a sum of 100 for a specific generation and attribute as confirmed by the sponsor. However, there is no constraint in the code for this. Not having a sum of 100 can give false results when estimating the `attributeProbabilityIndex` in this instance :

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L169

```js
    function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
        public 
        view 
        returns (uint256 attributeProbabilityIndex) 
    {
        uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute); 
        
        uint256 cumProb = 0;
        uint256 attrProbabilitiesLength = attrProbabilities.length; // 6 ? I guess not
        for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
@>          cumProb += attrProbabilities[i];
            if (cumProb >= rarityRank) { 
                attributeProbabilityIndex = i + 1;
                break;
            }
        }
        return attributeProbabilityIndex; 
    }
```

Morover, we have noticed that `attributeProbabilityIndex`which is used for the calculation of the `attributeIndex` in the function `createPhysicalAttributes` and gives the theritical value of the cosmetics of a fighter will always be the same for a specific generation and attribute. This seems wrong as the cosmetics of attribute of attribute should be somewhat randomized.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L108-L109

## [L-09] Unfair calculations in `RankedBattle::_addResultPoints`

A user that has very little `accumulatedPointsPerFighter` way less than the points required for the round it will not update his stake at Risk and he can get away with it.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L479

```js
  if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
                /// If the fighter has a positive point balance for this round, deduct points 
                points = stakingFactor[tokenId] * eloFactor;
                if (points > accumulatedPointsPerFighter[tokenId][roundId]) { //@audit if user has very little positive accumulatedPointsPerFighter he can get away with the loss 
                    points = accumulatedPointsPerFighter[tokenId][roundId];
                }
                accumulatedPointsPerFighter[tokenId][roundId] -= points;
                accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
                totalAccumulatedPoints[roundId] -= points;
                if (points > 0) {
                    emit PointsChanged(tokenId, points, false);
                }
```
If someone finds a way to always have some points during a round and before a battle he will make sure to never lose any funds.


## [NC-01] Player can spam initiate battles without staking anything. 

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L505

Players will spend voltage and not get any points but will accumulate wins or losses which will be written to the storage. 

## [NC-02] Remove unnecessary inherits

The `ERC721` contract does not need to be imported again if it is already imported in the `ERC721Enumerable` from Openzeppelin.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L16

```solidity
contract FighterFarm is ERC721, ERC721Enumerable {..}
```

## [NC-03] Redundant attributeProbabilities initialization in the `AiArenaHelper` contract

The line `addAttributeProbabilities(0, probabilities);` is doing the same as

```solidity
for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            ...
```

So adding the 2nd line inside the for loop is redundant.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41

```solidity
constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i]; //@audit does the exact same thing as the line before
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    }
```

## [NC-04] These variables can be defined as public instead of private
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L61C1-L67C28

```solidity
/// The address that has owner privileges (initially the contract deployer).
address _ownerAddress;

    /// Total number of game items.
    uint256 _itemCount = 0;

    /// @dev The Neuron contract instance.
    Neuron _neuronInstance;
```

## [NC-05] Better to use `hashstructs` for better readability and security

Since the function `claimFighters` is verifying that the correct batch of data are being submitted. it would better to create a struct containing all of these data and use them in a signature. This can be achieved through EIP-712.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L199-L205