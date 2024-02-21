	
## [G-01] Uint8[] array instead of string[]
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L17 
```
uint8[] public attributes = [0, 1, 2, 3, 4, 5];
// Instead of
string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"];
```
##### Gas Saving:
* 216851 gas during deployment
* 303 gas every update on attributeProbabilities , attributeToDnaDivisor
More Changes:
```
   mapping(uint256 => mapping(uint8 => uint8[])) public attributeProbabilities;
   mapping(uint8 => uint8) public attributeToDnaDivisor;
```

## [G-02]Define and use defaultAttributeDivisor inside constructor
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L20 
```
// Use 
uint8[] memory defaultAttributeDivisor = [2, 3, 5, 7, 11, 13]; 
// instead of 
uint8[] public defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];
```
* Inside constructor to save gas.
* Gas Saving: 98006 gas
* Constructor looks like this, after changes
```
constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;
        uint256 attributesLength = attributes.length; 
        uint8[6] memory defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];     
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```

## [G-03]Repeat Operation of writing to attributeProbabilities[0][attributes[i]] 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L49 
```
// Remove this
addAttributeProbabilities(0, probabilities);
```

* Is redundant because attributeProbabilities is initialized in line 53 also.

* Gas Saving: 94140 gas (minimum)


## [G-04]Use ‘6’ instead of Attributes.length to save gas(2100)

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L47 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L70 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L72 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L96 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L98 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L135 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L147 
* attributes.length is fixed 

## [G-05]Remove Unnecessary Condition
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L154 
```
require(
           allGameItemAttributes[tokenId].finiteSupply == false ||  
           (
               allGameItemAttributes[tokenId].finiteSupply == true &&
               quantity <= allGameItemAttributes[tokenId].itemsRemaining
           )
       );
```

1) The first part of the OR statement checks if allGameItemAttributes[tokenId].finiteSupply is false. If this condition is met, the entire statement evaluates to true, and the nested condition is not checked.
2) If allGameItemAttributes[tokenId].finiteSupply is true.The first part of the OR statement checks that allGameItemAttributes[tokenId].finiteSupply is not false. 
So we need not check again in the second part of OR that allGameItemAttributes[tokenId].finiteSupply is true

* Code after Optimization
```
require(
           allGameItemAttributes[tokenId].finiteSupply == false ||  
               quantity <= allGameItemAttributes[tokenId].itemsRemaining
       );
```


## [G-06] Uint8 instead of uint256 for MergingPool
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L324 
```
   function updateBattleRecord(
       uint256 tokenId,
       uint256 mergingPortion, //@note Could be uint8
       uint8 battleResult,
       uint256 eloFactor,
       bool initiatorBool
   )
       external
   {  require(msg.sender == _gameServerAddress);
  require(mergingPortion <= 100);
```
* Since it is required that mergingPool<=100 => We could use uint8 for mergingpool instead of uint256
* Similar changes in this function also
```
   function _addResultPoints(
       uint8 battleResult,
       uint256 tokenId,
       uint256 eloFactor,
       uint256 mergingPortion,
       address fighterOwner
   )
```

## [G-07]Use uint8 instead of uint16
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L92 
```
function viewFighterInfo(
       Fighter storage self,
       address owner
   )
       public
       view
       returns (
           address,
           uint256[6] memory,
           uint256,
           uint256,
           string memory,
           string memory,
           uint16         // Use uint8 instead of uint16
       )
   {
       return (
           owner,
           getFighterAttributes(self),
           self.weight,
           self.element,
           self.modelHash,
           self.modelType,
           self.generation
       );
   }
```
* Self.generation is uint8
* Extra 7 gas to convert from uint8 to uint16

## [G-08] Use _itemCount instead of allGameItemAttributes.length to save gas
```
   function uniqueTokensOutstanding() public view returns (uint256) {
       return allGameItemAttributes.length; // @note why not return _itemCount ?
   }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L286C1-L286C45 
