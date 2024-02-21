
# Gas Report : AI Arena Contest | 9th Feb 2024 - 21st Feb 2024

## Executive summary
 **AI Arena** is a PvP platform fighting game where the fighters are AIs that were trained by humans. In AI Arena you train an AI character to battle in a platform fighting game. Imagine a cross between Pokémon and Super Smash Bros, but the characters are AIs, and you can train them to learn almost any skill in preparation for battle.

## Overview

| **Project Name**	| **AI Arena**                                            |
| --------------------- | ------------------------------------------------------- |
| **Repository**   	| [Github](https://github.com/code-423n4/2024-02-ai-arena)|
| **Website**      	| [AI Arena](https://aiarena.io/#/)                       |
| **X(Twitter)**   	| [𝕏](https://twitter.com/aiarena_)             	  |
| **Discord**      	| [Discord](https://discord.gg/aiarena)                   |
| **Total SLoC**  	| **1197** over **8** contracts                           |

---

## List of Findings 

|   **ID**              | **Title**                                                                                                | 
|:--------------------:|---------------------------------------------------------------------------------------------------|  
| [G-01](#g-01-check-for-amount--0-before-making-_burn-call) | Check for `amount = 0` before making `_burn()` call                     |  
| [G-02](#g-02-cache-attributeslength-only-once-to-avoid-extra-gas-usage) | Cache `attributes.length` only once to avoid extra gas usage |  
| [G-03](#g-03-redundant-operation-of-initializing-attributeProbabilities-mapping-in-constructor-leads-to-unnecessary-gas-consumption) |  Redundant operation of initializing `attributeProbabilities` mapping in `constructor` leads to unnecessary gas consumption            |  


---
---
## [G-01] Check for `amount = 0` before making `_burn()` call
#### Description:
It is unnecessary to call `_burn()`  on amount  = 0, it leads to wastage of gas as `_burn()`  itself doesn't have such check before writing to state variables of the contract, thereby calling `SSTORE` EVM opcode, which costs significant amount of gas.

*"SSTORE, the opcode which saves data in storage, costs 20,000 gas when a storage value is set to a non-zero value from zero and costs 5000 gas when a storage variable’s value is set to zero or remains unchanged from zero."*

 [Neuron.sol::burn()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L163-L165)
 
**Code:**
```javascript
function  burn(uint256 amount) public virtual {
	//gas- don't call for amt = 0, wastage of gas on add/sub 0 
	//to/from state variable.
	_burn(msg.sender, amount);
}
```
#### Recommendation: 
Updating the above code snippet to the below given can avoid unnecessary usage of excessive gas caused by absurd operations.
```javascript
function  burn(uint256 amount) public virtual {
	 require(amount!=0,"Zero Amount not allowed")
	_burn(msg.sender, amount);
}
```
##
## [G-02] Cache `attributes.length` only once to avoid extra gas usage
#### Description: 
The `attributesLength` caches to avoid computation of array length of state variable (costly) with each iteration is appreciated.

But, if we declare it before doing `require(attributeDivisors.length == attributes.length);`
it can save some gas, though we might introduce `mstore` opcode use, but since the function is only callable by owner, it is less likely that above require check to fail. 

 [AiArenaHelper::addAttributeDivisor](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L68-L76)

**Code:** 
 ```javascript
 function addAttributeDivisor(uint8[] memory attributeDivisors) external {
        require(msg.sender == _ownerAddress);
        require(attributeDivisors.length == attributes.length);
	
        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
        }
    }    
```

#### Recommendation: 
```diff
function addAttributeDivisor(uint8[] memory attributeDivisors) external {
+ 	uint256 attributesLength = attributes.length;
        require(msg.sender == _ownerAddress);
        require(attributeDivisors.length == attributesLength);

- 	uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i];
        }
}    
```
### Another Instance: 
 [AiArenaHelper.sol::createPhysicalAttributes](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L96-L98)

**Code:**
```javascript
uint256[] memory finalAttributeProbabilityIndexes =  new  uint[](attributes.length);

uint256 attributesLength = attributes.length;
```
#### Recommendation: 
```diff
+ uint256 attributesLength = attributes.length;
  uint256[] memory finalAttributeProbabilityIndexes =  new  uint[](attributes.length);
- uint256 attributesLength = attributes.length;
```
##
## [G-03] Redundant operation of initializing `attributeProbabilities` mapping in `constructor` leads to unnecessary gas consumption  
#### Description: 
In the `constructor` while initializing state variables, we make `addAttributeProbabilities(0, probabilities)` function call which has the following function body:

**Code:** 
[AiArenaHelper.sol::addAttributeProbabilities()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131-L139)
```javascript
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
    require(msg.sender == _ownerAddress);
    require(probabilities.length == 6, "Invalid number of attribute arrays");
// here we have generation as 0
    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
        attributeProbabilities[generation][attributes[i]] = probabilities[i];
    }
}
```
But our constructor does this same operation after calling the above function, following is the code from the `constructor`:
[AiArenaHelper.sol::constructor](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41-L52)
```javascript
uint256 attributesLength = attributes.length;
for (uint8 i = 0; i < attributesLength; i++) {
    attributeProbabilities[0][attributes[i]] = probabilities[i];
    attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
}
```
This causes additional gas usage for doing an operation that has been done again, the modification of the state variables by extra call will not be done. Just the contract deployer will be charged additional gas for doing redundant initialization of the mapping. 
#### Recommendation: 
Based on the implementation of the `addAttributeProbabilities()` it appears that, it just initializes the attributeProbabilities mapping, but in the constructor `for` loop we also initialize the attributeToDnaDivisor mapping. So `addAttributeProbabilities()` and `addAttributeDivisor()` should be called in `constructor` or either keep the `for` loop in the constructor which does the initialization for both mappings. Thus, include initialization only once, either via function call, or initializing in the `constructor` itself to avoid unnecessary usage of gas.  
***
---