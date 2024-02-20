
# Low

## [L-1] Dangerous one step ownership transfer functions

### Impact

The `transferOwnership` function implemented in the following contracts: `AAMintPass`, `AiArenaHelper`,`FighterFarm`, `GameItems`, `MergingPool`, `Neuron`, `RankedBattle` and `VoltageManager` should be considered to be two phase instead of one step. With the current implementation ownership transfer to an account being invalid will cause the contracts to stay without an owner indefintitely, blocking the majority of functionality.

```javascript
 function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
```
### Recommended Mitigation
Make use of OpenZeppelin's `Ownable` contract or implement `acceptOwnership` function that makes transferring ownership a two-step process.

## [L-2] `Neuron::mint` function require statement can be changed to allow MAX_SUPPLY of tokens
With the current require statement implementation, the `MAX_SUPPLY` can't be strictly reached, only closely approached due to "<" operator instead of "<=".

### Recommended Mitigation
```diff
-require(totalSupply() + amount < MAX_SUPPLY,"Trying to mint more than the max supply");
+require(totalSupply() + amount <= MAX_SUPPLY,"Trying to mint more than the max supply");
```
## [L-3] Lack of event emitted after changing a global variable

Consider emitting an event in `StakeAtRisk::setNewRound` after `roundId = roundId_;` is successfully set.

```diff
+event RoundIdChanged(uint256 newRoundId);
___
bool success = _sweepLostStake();
        if (success) {
            roundId = roundId_;
+           emit RoundIdChanged(roundId);
        }
```

## [L-4] Performing the same variable assignment operation twice

In `AiArenaHelper::constructor` the `addAttributeProbabilities` function is called where this loop is executed:

```javascript
for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
```

Then, back in the constructor this loop is executed:

```javascript
for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
```

Essentially asigning `attributeProbabilities[0][attributes[i]]` to `probabilities[i]` twice.

## [L-5] Missing emit for a declared event
The `Neuron::TokensMinted` event is being declared, but never emitted. This may cause reduced off-chain monitoring capabilities.

# Info/Gas

## [I-1] Possible mistake in an error message

The following error message in `RankedBattle::setNewRound` is likely not intended:

```javascript
require(totalAccumulatedPoints[roundId] > 0, "b");
```

## [I-2] Lack of check if the token exists in `FighterFarm::tokenURI`

Consider adding a check included in the original ERC721 implementation of `tokenURI` function:

```diff
function tokenURI(
        uint256 tokenId
    ) public view override(ERC721) returns (string memory) {
+       _requireMinted(tokenId);
        return _tokenURIs[tokenId];
    }
```


## [I-3] Use of modifiers instead of code duplication

Consider using modifiers instead of duplicating code for `require`:

```diff
-   require(msg.sender == _rankedBattleAddress, "Not authorized to set new round");
+   modifier onlyRankedBattle {
+      require(msg.sender == _rankedBattleAddress, "Not authorized to set new round");
+   }
```

```diff
-   require(msg.sender == _ownerAddress);
+   modifier onlyOwnerAddress {
+      require(msg.sender == _ownerAddress);
+   }
```

etc.

## [G-1] Addresses that should be made `immutable`

The following addresses should be `immutable` to minimize gas use:
`FighterFarm::treasuryAddress`,
`Neuron::treasuryAddress`,
`StakeAtRisk::treasuryAddress`,
`StakeAtRisk::_rankedBattleAddress`,
`MergingPool::_rankedBattleAddress`,


## [G-2] Many instances of assigning a variable that are obsolete

All instances of the following assignment:`uint256 attributesLength = attributes.length;` and `uint256 recipientsLength = recipients.length;` are unnecessary and they cause a small gas waste.


### Time spent:
6 hours