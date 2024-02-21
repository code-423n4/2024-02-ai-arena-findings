# AI Arena protocol QA Report


## [L-01] Hardcoded cost of rerolls and prices of batteries.
### Context
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L39
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L208-L235

### Description
In the FighterFarm.sol contract, the prices for rerolls are hardcoded as constant values. Prices for items in the GameItems.sol contract are also hardcoded at the time of the creation of an item. Hardcoding prices can lead to inflexibility in adjusting prices based on market conditions or game dynamics. Also the price of the NRN token used for payments is a subject to volatility.
Relevant code:
```solidity
uint256 public rerollCost = 1000 * 10**18;    
```
```solidity
struct GameItemAttributes {
        string name;
        bool finiteSupply;
        bool transferable;
        uint256 itemsRemaining;
        uint256 itemPrice;
        uint256 dailyAllowance;
    }  
```
### Recommendation 
To introduce flexibility and adaptability into the pricing mechanism, it's recommended to refactor the code to allow for dynamic pricing adjustments.
```solidity
function setRerollPrice(uint256 newPrice) external onlyOwner {
    require(newPrice > 0, "Price must be greater than zero");
    rerollCost  = newPrice;
}
```
```solidity
function setItemPrice(uint256 id,uint256 newPrice) external onlyOwner {
    require(newPrice > 0, "Price must be greater than zero");
    allGameItemAttributes[id].itemPrice = newPrice;
}
```
## Title [L-02] `updateModel()` possible overflow.
### Context
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L283-L295
### Description
The issue arises from the fact that `numTrained` is capped at uint32 for each NFT, while `totalNumTrained` is also a uint32 but represents the sum of all training sessions across all NFTs. This design flaw can lead to unexpected behavior and potential overflow scenarios.
Relevant code:
```solidity
/// @notice Aggregate number of training sessions recorded.
uint32 public totalNumTrained;
/// @notice Mapping of tokenId to number of times trained.
mapping(uint256 => uint32) public numTrained;
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
    }
```
For instance, if one NFT undergoes the updateModel function for uint32.max times, both its `numTrained` and the `totalNumTrained` will reach uint32.max. Subsequently, if another NFT attempts to execute the `updateModel()` function, the addition of one more training session would cause an overflow, as totalNumTrained cannot accommodate a value greater than uint32.max.
### Recommendation 
One potential solution could involve using larger integer types for tracking training session counts across all NFTs, such as uint256, to accommodate a larger number of training sessions.

## [L-03] Potential Revert Due to Out-of-Gas Error in `claimNRN()`.
### Context
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L294-L311
### Description
`claimNRN()` function allows users to claim rewards for multiple rounds.
Relevant code:
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
            numRoundsClaimed[msg.sender] += 1;
        }
        if (claimableNRN > 0) {
            amountClaimed[msg.sender] += claimableNRN;
            _neuronInstance.mint(msg.sender, claimableNRN);
            emit Claimed(msg.sender, claimableNRN);
        }
    }
```
The problem arises when a user does not claim rewards for too many rounds, potentially leading to the state where this function reverts due to out-of-gas error.
### Recommendation 
Allow users to specify the number of rounds they want to claim for.

## Title [L-04] `claimFighters` possible overflow.
### Context
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L191-L222
### Description
`claimFighters()` function allows users to claim a pre-determined number of fighters.
It does so by minting NFTs via a loop. The length of the loop is determined by the total amount of NFTs to mint.
Relevant code:
```solidity
 uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
```
The potential overflow exists in this line of code. As both `numToMint` variables are of type uint8 if summation overflows `type(uint8).max`, the transaction will revert.
### Recommendation
Change calculations to:
```solidity
uint16 totalToMint = uint16(numToMint[0]) + uint16(numToMint[1]);
```
This way, both variables will first be typecasted to `uint16` eliminating the possibility of an overflow.

## Title [L-05] Weak source of randomness for fighter's DNA.
### Context
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L212-L221
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L313-L331
### Description
DNA is used to determine rare physical attributes in `AiArenaHelper.createPhysicalAttributes()`.
The problem is that when a fighter is created through `claimFighters()` or `mintFromMergingPool()` function, DNA is just a hash of `msg.sender` and the length of the fighters array.
Relevant code:
```solidity
_createNewFighter(
                msg.sender, 
                uint256(keccak256(abi.encode(msg.sender, fighters.length))),
                modelHashes[i], 
                modelTypes[i],
                i < numToMint[0] ? 0 : 1,
                0,
                [uint256(100), uint256(100)]
            );
```
This introduces a potential for physical attributes manipulation, as a user will have the ability to decide when to call those functions, deciding what DNA he will receive based on the length of the fighters array.
### Recommendation
Introduce a stronger source of randomness, like Chainlink VRF.

## Title [L-06] `MergingPool.getFighterPoints()` incorrectly initializes an array.
### Context
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L205-L211
### Description
`getFighterPoints()` is used to retrieve the points for multiple fighters up to the specified maximum token ID. This information will be used in determining winners of the current round.
Relevant code:
```solidity
function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
        uint256[] memory points = new uint256[](1);
        for (uint256 i = 0; i < maxId; i++) {
            points[i] = fighterPoints[i];
        }
        return points;
    }
```
The problem is that the `points` array is initialized incorrectly, leading to a revert, when someone tries to query points for `maxId` greater than 1.
This will create issues in determining the winner, as querying each fighter manually can quickly become a burdensome action.
### Recommendation
Change the code to:
```solidity
int256[] memory points = new uint256[](maxId);
```

## [L-07] `claimRewards()` does not validate custom attributes.
### Context
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139-L167
### Description
`claimRewards()` function of the MergingPool allows user to claim their rewards in form of NFTs.
The problem is that it does not validate `customAttributes` parameter, which is user-supplied.
```solidity
function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
```
Thos custom attributes are just assigned to the NFT in `_createNewFighter()` function
```solidity
} else {
            element = customAttributes[0];
            weight = customAttributes[1];
            newDna = dna;
        }
```
### Recommendation
Validate that attributes provided by the user are in the expected range of the system.