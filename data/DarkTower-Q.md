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

## [L-02] Use similar ownership transfers to Ownable2Step

Ownership transfers happen across 7 instances in the in-scope contracts. These transfers are carried out by the contract owner address referenced by `_ownerAddress` state variable and happen as a one-way transaction flow. This is a flaw in the sense that such transfer have a higher likelihood of the owner being wrongfully changed which creates a single-point failure. For added security, it is better to implement a two-step ownership transfer similar to the Openzeppelin `Ownable2Step` contract ownership transfer.

One of the single-step ownership transfer is referenced below in the `AiArenaHelper` contract line 61:

```js
/// @notice Transfers ownership from one address to another.
    /// @dev Only the owner address is authorized to call this function.
    /// @param newOwnerAddress The address of the new owner
@>  function transferOwnership(address newOwnerAddress) external {
      require(msg.sender == _ownerAddress);
      _ownerAddress = newOwnerAddress;
    }
```

> All 7 instances

- (AiArenaHelper.sol#L61)[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L61]
- (FighterFarm.sol#L120)[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L120]
- (GameItems.sol#L108)[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L108]
- (MergingPool.sol#L89)[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L89]
- (Neuron.sol#L85)[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L85]
- (RankedBattle.sol#L167)[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L167]
- (VoltageManager.sol#L64)[https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L64]

## [L-03] Players can burn 1 full battery even at 90% voltage.

The function is called when when a voltage battery to replenish voltage for a player in AI Arena. But it allows players to burn his battery even at 90% voltage.

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

## [L-04] Players can only claim one of two signatures given by the server using `claimFighters`.

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

## [L-05] Redundant attributeProbabilities initialisation in the `AiArenaHelper` contract

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

## [L-06] Player can spend any amount of voltage to replenish and lose `VoltageReplenishTime` when player try to replenish even if his `ownerVoltage` is high.

## [L-07] Use Oppenzeppelin's `erecover` function to avoid signature replay due to duplicate v value

The built-in EVM precompile ecrecover is susceptible to signature malleability (because of non-unique s and v values) which could lead to replay attacks. We recommend to use Oppenzeppelin's `erecover` function to avoid signature replay due to duplicate v value.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L206

```solidity
require(Verification.verify(msgHash, signature, _delegatedAddress)); // @audit better to use hashstruct for signing data for better readability and security
```

## [L-08] These variables can be defined as public instead of private.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L61C1-L67C28

```solidity
/// The address that has owner privileges (initially the contract deployer).
address _ownerAddress;

    /// Total number of game items.
    uint256 _itemCount = 0;

    /// @dev The Neuron contract instance.
    Neuron _neuronInstance;
```

## [L-09] Approval isn't revoked from the contract to spend user's funds if the `transferFrom` fails.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L376
```solidity 
function reRoll(uint8 tokenId, uint8 fighterType) public { //@note can you give this any fighter type that you want ?
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

        _neuronInstance.approveSpender(msg.sender, rerollCost); // @note approves this contract
@>      bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost); // @audit should use safetransferFrom

```

## [NC-01] Remove unnecessary inherits
The `ERC721` contract does not need to be imported again if it is already imported in the `ERC721Enumerable` from Openzeppelin. 
 
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L16
```solidity
contract FighterFarm is ERC721, ERC721Enumerable {..}
```