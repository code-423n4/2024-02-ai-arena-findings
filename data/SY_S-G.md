## Summary

### Gas Optimization

no |Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |Mappings not used externally/internally can be marked private  |  |17|--| 
| [G-02] |Redundant state variable getters |  |15|--| 
| [G-03] |Use named returns for local variables of pure/view functions where it is possible |  ||--| 
| [G-04] |Using PRIVATE rather than PUBLIC FOR Constants/Immutable, Saves Gas|  |8|--| 
| [G-05] |Before transfer of  some functions, we should check some variables for possible gas save |  |2|--| 
| [G-06] |Require() or revert() statements that check input arguments should be at the top of the function |  |3|--| 
| [G-07] |Optimize Address Storage Value Management with `assembly` |  |17|--| 
| [G-08] |Use assembly to emit events |  |19|--| 
| [G-09] |Use byte32 in place of string|  |21|--| 
| [G-10] |Divisions which do not divide by -X cannot overflow or overflow so such operations can be unchecked to save gas |  |8|--| 
| [G-11] |Do not calculate constants |  |3|--| 
| [G-12] |Stack variable cost less while used in emiting event|  |8|--| 
| [G-13] | `internal` functions only called once can be inlined to save gas |  |2|--| 
| [G-14] |Use shift Left instead of multiplication if possible to save gas |  |3|--| 




## Gas Optimizations  

## [G-1] Mappings not used externally/internally can be marked private 
in this case, since all mappings are declared as public, they can be accessed and modified by any external contract or account. If you do not need this level of accessibility, it's recommended to mark them as private to restrict their usage to within the contract where they are defined. This can help prevent unintended or malicious manipulation of the mapping data from external sources.


```solidity
file:/src/AiArenaHelper.sol
 30   mapping(uint256 => mapping(string => uint8[])) public attributeProbabilities;
 33   mapping(string => uint8) public attributeToDnaDivisor;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/AiArenaHelper.sol#L30


```solidity
File: /src/FighterFarm.sol
  76  mapping(uint256 => bool) public fighterStaked;

  79   mapping(uint256 => uint8) public numRerolls;

  82  mapping(address => bool) public hasStakerRole;

 
  85   mapping(uint8 => uint8) public numElements;

    
  88  mapping(address => mapping(uint8 => uint8)) public nftsClaimed;


  91  mapping(uint256 => uint32) public numTrained;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L76


```solidity
file: /src/GameItems.sol
  74  mapping(address => mapping(uint256 => uint256)) public allowanceRemaining;

   
  77  mapping(address => mapping(uint256 => uint256)) public dailyAllowanceReplenishTime;

   
  80  mapping(address => bool) public allowedBurningAddresses;

    
  83  mapping(address => bool) public isAdmin;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/GameItems.sol#L74
```solidity
file:
    48 	  mapping(address => uint32) public numRoundsClaimed;
	
    
    51 	  mapping(uint256 => uint256) public fighterPoints;

  
    54	  mapping(uint256 => address[]) public winnerAddresses;    

    
    57     mapping(uint256 => bool) public isSelectionComplete;

   
    60     mapping(address => bool) public isAdmin;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/MergingPool.sol#L48-L60
## [G-2] Redundant state variable getters
Getters for public state variables are automatically generated by the solidity compiler so there is no need to code them manually as this increases deployment cost.


File: /src/FighterFarm.sol
  76  mapping(uint256 => bool) public fighterStaked;

  79   mapping(uint256 => uint8) public numRerolls;

  82  mapping(address => bool) public hasStakerRole;

 
  85   mapping(uint8 => uint8) public numElements;

    
  88  mapping(address => mapping(uint8 => uint8)) public nftsClaimed;


  91  mapping(uint256 => uint32) public numTrained;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L76


```solidity
file: /src/GameItems.sol
  74  mapping(address => mapping(uint256 => uint256)) public allowanceRemaining;

   
  77  mapping(address => mapping(uint256 => uint256)) public dailyAllowanceReplenishTime;

   
  80  mapping(address => bool) public allowedBurningAddresses;

    
  83  mapping(address => bool) public isAdmin;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/GameItems.sol#L74
```solidity
file:
    48 	  mapping(address => uint32) public numRoundsClaimed;
	
    
    51 	  mapping(uint256 => uint256) public fighterPoints;

  
    54	  mapping(uint256 => address[]) public winnerAddresses;    

    
    57     mapping(uint256 => bool) public isSelectionComplete;

   
    60     mapping(address => bool) public isAdmin;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/MergingPool.sol#L48-L60
## [G-3] Use named returns for local variables of pure/view functions where it is possible
 

```solidity

file:src/AiArenaHelper.sol
  157    function getAttributeProbabilities(uint256 generation, string memory  158 attribute) 
  159     public 
  160     view 
  161    returns (uint8[] memory) 
      {
  163    return attributeProbabilities[generation][attribute];
    }    


```

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/AiArenaHelper.sol#L157-L163
```solidity
file: src/FighterFarm.sol
 129  function incrementGeneration(uint8 fighterType) external returns (uint8) {
 299   function doesTokenExist(uint256 tokenId) external view returns (bool) {
 395    function contractURI() public pure returns (string memory) {
 402    function tokenURI(uint256 tokenId) public view override(ERC721) returns (string memory) {
 410    function supportsInterface(bytes4 _interfaceId)
 411      public
 412     view
 413     override(ERC721, ERC721Enumerable)
 414    returns (bool)
 39  function _ableToTransfer(uint256 tokenId, address to) private view returns(bool) {
```
https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L129-L129


## [G-4] Using PRIVATE rather than PUBLIC FOR Constants/Immutable, Saves Gas        
If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```solidity
file:/src/FighterFarm.sol
 33   uint8 public constant MAX_FIGHTERS_ALLOWED = 10;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L33

```solidity
file:Neuron.sol
28    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");


    /// @notice Role for spending tokens.
31  bytes32 public constant SPENDER_ROLE = keccak256("SPENDER_ROLE");
    
    /// @notice Role for staking tokens.
34    bytes32 public constant STAKER_ROLE = keccak256("STAKER_ROLE");


    /// @notice Initial supply of NRN tokens to be minted to the treasury.
37  uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;


    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
40   uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;


    /// @notice Maximum supply of NRN tokens.
43    uint256 public constant MAX_SUPPLY = 10**18 * 10**9;


```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L28-L44


```solidity
file:/src/RankedBattle.sol
53  uint8 public constant VOLTAGE_COST = 10;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L53

## [G-5] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:


```solidity
file:/src/FighterFarm.sol
451        super._beforeTokenTransfer(from, to, tokenId);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L451

```solidity
file:/src/StakeAtRisk.sol
143       return _neuronInstance.transfer(treasuryAddress, totalStakeAtRisk[roundId]);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L143C4-L143

## [G-6] Require() or revert() statements that check input arguments should be at the top of the function  
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.
```solidity
file:
206       require(Verification.verify(msgHash, signature, _delegatedAddress));

208        require(modelHashes.length == totalToMint && modelTypes.length == 
totalToMint);

250     require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L206C6-L206

## [G-7] Optimize Address Storage Value Management with `assembly`

*There are 17 instance(s) of this issue:*


```solidity

file:/src/AiArenaHelper.sol
42       _ownerAddress = msg.sender;
63        _ownerAddress = newOwnerAddress;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L42

```solidity
file:src/FighterFarm.sol
 122       _ownerAddress = newOwnerAddress;
 173         _mergingPoolAddress = mergingPoolAddress;
 182         _tokenURIs[tokenId] = newTokenURI;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L122
```solidity
file:/src/GameItems.sol
110       _ownerAddress = newOwnerAddress;
119         isAdmin[adminAddress] = access;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L110
```solidity
file:/src/MergingPool.sol
91       _ownerAddress = newOwnerAddress;
100        isAdmin[adminAddress] = access;
108        winnersPerPeriod = newWinnersPerPeriodAmount;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L91
```solidity
file:/src/Neuron.sol
 87       _ownerAddress = newOwnerAddress;
 120        isAdmin[adminAddress] = access;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L87
```solidity
file:/src/RankedBattle.sol
 169       _ownerAddress = newOwnerAddress;
 186        _gameServerAddress = gameServerAddress;
 194        _stakeAtRiskAddress = stakeAtRiskAddress;
 203        _neuronInstance = Neuron(nrnAddress);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L169
```solidity
file:/src/VoltageManager.sol
 66       _ownerAddress = newOwnerAddress;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L66
## [G-8] Use assembly to emit events
We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

*There are 19  instance(s) of this issue:*
```solidity
file:/src/FighterFarm.sol
272          emit Locked(tokenId);
274           emit Unlocked(tokenId);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L272
```solidity
file:/src/FighterOps.sol
61       emit FighterCreated(id, weight, element, generation);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L61
```solidity
file:
 130         emit Unlocked(tokenId);
 132         emit Locked(tokenId);
 174         emit BoughtItem(msg.sender, tokenId, quantity);
 231          emit Locked(_itemCount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L130
```solidity
file:MergingPool.sol
165          emit Claimed(msg.sender, claimIndex);
199        emit PointsAdded(tokenId, points);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L165
```solidity
file:/src/Neuron.sol
 144       emit TokensClaimed(msg.sender, amount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L144
```solidity
file:/src/RankedBattle.sol
263          emit Staked(msg.sender, amount);
288            emit Unstaked(msg.sender, amount);
309            emit Claimed(msg.sender, claimableNRN);
470                emit PointsChanged(tokenId, points, true);
489                    emit PointsChanged(tokenId, points, false);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L263
```solidity

file:/src/StakeAtRisk.sol
 105           emit ReclaimedStake(fighterId, nrnToReclaim);
 126        emit IncreasedStakeAtRisk(fighterId, nrnToPlaceAtRisk);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L105
```solidity
file:/src/VoltageManager.sol
 98       emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
 111        emit VoltageRemaining(spender, ownerVoltage[spender]);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L98

## [G-9] Use byte32 in place of string

For strings of 32 char strings and below you can use bytes32 instead as it's more gas efficient

*There are 21 instance(s) of this issue:*
```solidity
file:/src/AiArenaHelper.sol
17     string[] public attributes = ["head", "eyes", "mouth", "body", "hands""feet"];

33      mapping(string => uint8) public attributeToDnaDivisor;
  
157   function getAttributeProbabilities(uint256 generation, string memory attribute) 

169     function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 


https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L17

```solidity
file:/src/FighterFarm.sol
315         string calldata modelHash, 
316        string calldata modelType, 
395    function contractURI() public pure returns (string memory) {
402    function tokenURI(uint256 tokenId) public view override(ERC721) returns (string memory) {

431            string memory,
432           string memory,
487       string memory modelHash,
488      string memory modelType, 
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L315-L316

```solidity
file:/src/FighterOps.sol
 41       string modelHash;
 42       string modelType;
 90          string memory,
 91          string memory,
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L41-L42

```solidity
file:/src/GameItems.sol
 49   string public name = "AI Arena Game Items";

 52   string public symbol = "AGI";

 194    function setTokenURI(uint256 tokenId, string memory _tokenURI) public {

 249    function contractURI() public pure returns (string memory) {

 256    function uri(uint256 tokenId) public view override returns (string memory) {

```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L49-L53

## [G-10] Divisions which do not divide by -X cannot overflow or overflow so such operations can be unchecked to save gas
Make such found divisions are unchecked when ensured it is safe to do so
*There are 8 instance(s) of this issue:*
```solidity
file:/src/RankedBattle.sol
 302               accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
 303           ) / totalAccumulatedPoints[currentRound];

 393                 accumulatedPointsPerAddress[claimer][i] * nrnDistribution
 394          ) / totalAccumulatedPoints[i];

 439        curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;

 449           uint256 mergingPoints = (points * mergingPortion) / 100;
```
527       uint256 stakingFactor_ = FixedPointMathLib.sqrt(
528         (amountStaked[tokenId] + stakeAtRisk) / 10**18
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L302-L303

## [G-11] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

*There are 3 instance(s) of this issue:*
```solidity
fille:/src/Neuron.sol
37     uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;


    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
40   uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;


    /// @notice Maximum supply of NRN tokens.
43   uint256 public constant MAX_SUPPLY = 10**18 * 10**9;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L37-L44

## [G-12] Stack variable cost less while used in emiting event
Even if the variable is going to be used only one time, caching a state variable and use its cache in an emit would help you reduce the cost by at least ***9 gas***

*There are 8 instance(s) of this issue:*

```solidity
file:/src/FighterOps.sol
61        emit FighterCreated(id, weight, element, generation);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L61

```solidity
file:/src/GameItems.sol
174           emit BoughtItem(msg.sender, tokenId, quantity);
231          emit Locked(_itemCount);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L174
```solidity
file:/src/MergingPool.sol
            
 199       emit PointsAdded(tokenId, points);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L199

```solidity
file:/src/Neuron.sol
 144       emit TokensClaimed(msg.sender, amount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L144

```soldity
file:/src/RankedBattle.sol
 263           emit Staked(msg.sender, amount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L263

```solidity
file:/src/StakeAtRisk.sol
 126       emit IncreasedStakeAtRisk(fighterId, nrnToPlaceAtRisk);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L126

```solidity
file:/src/VoltageManager.sol
98        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L98
## [G-13]  `internal` functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

*There are 2 instance(s) of this issue:*


```solidity
file:/src/FighterFarm.sol
 247     function _beforeTokenTransfer(address from, address to, uint256 tokenId)
 248       internal

```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L447-L448

## [G-14]Use shift Left instead of multiplication if possible to save gas
*There are 3 instance(s) of this issue:*
```solidity
file:/src/Neuron.sol#
    uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;


    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
    uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;


    /// @notice Maximum supply of NRN tokens.
    uint256 public constant MAX_SUPPLY = 10**18 * 10**9;


```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L37-L44