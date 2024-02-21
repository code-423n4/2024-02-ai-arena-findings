## \[L-01\] Stop Using Solidity's transfer() Now

**Proof Of Concept**  
https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

```
bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L100

```
return _neuronInstance.transfer(treasuryAddress, totalStakeAtRisk[roundId]);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L143

## \[L-02\] Insufficient coverage

Description  
The test coverage rate of the project is ~95%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## \[L-03\] Use of magic numbers is confusing and risky

### Summary:

Magic numbers are hardcoded numbers used in the code which are ambiguous to their intended purpose. These should be replaced with constants to make code more readable and maintainable.

### Details:

Values are hardcoded and would be more readable and maintainable if declared as a constant

### Github Permalinks

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L107

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L499

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L332

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L449

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L118

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L93C4-L99C6

### Mitigation

Replace magic hardcoded numbers with declared constants.

## \[N-01\] Missing reason strings in `require` statements

`Require` statements should have descriptive reason strings

```
File: src/AiArenaHelper.sol

62:  require(msg.sender == _ownerAddress);
69:  require(msg.sender == _ownerAddress);
132:  require(msg.sender == _ownerAddress);
145:  require(msg.sender == _ownerAddress);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L62

```
File: src/FighterFarm.sol

121: require(msg.sender == _ownerAddress);
130: require(msg.sender == _ownerAddress);
140: require(msg.sender == _ownerAddress);
148: require(msg.sender == _ownerAddress);
156: require(msg.sender == _ownerAddress);
164: require(msg.sender == _ownerAddress);
172: require(msg.sender == _ownerAddress);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L121

```
File: src/GameItems.sol

109: require(msg.sender == _ownerAddress);
118: require(msg.sender == _ownerAddress);
127: require(msg.sender == _ownerAddress);
140: require(msg.sender == _ownerAddress);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L109

```
File: src/MergingPool.sol

90: require(msg.sender == _ownerAddress);
99: require(msg.sender == _ownerAddress);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L90

```
File: src/Neuron.sol

86: require(msg.sender == _ownerAddress);
94: require(msg.sender == _ownerAddress);
102: require(msg.sender == _ownerAddress);
110: require(msg.sender == _ownerAddress);
119: require(msg.sender == _ownerAddress);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L86

```
File: src/RankedBattle.sol

168: require(msg.sender == _ownerAddress);
177: require(msg.sender == _ownerAddress);
185: require(msg.sender == _ownerAddress);
193: require(msg.sender == _ownerAddress);
202: require(msg.sender == _ownerAddress);
210: require(msg.sender == _ownerAddress);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L168

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L148

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L186

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L195

&nbsp;

## \[N-02\] For multiple lines use NetSpec Comment for better readability 

```
-	/// @title StakeAtRisk
-	/// @author ArenaX Labs Inc.
-	/// @notice Manages the NRNs that are at risk of being lost during a round of competition

+	/**
+    *  @title StakeAtRisk
+    *  @author ArenaX Labs Inc.
+    *  @notice Manages the NRNs that are at risk of being lost during a round of competition
+    * 
+    */
```

> All contest 

## \[N-03\] NatSpec comments should be increased in contracts

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[N-04\] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## \[N-05\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-06\] Function writing that does not comply with the Solidity Style Guide

Context  
All Contracts

Description  
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor  
receive function (if exists)  
fallback function (if exists)  
external  
public  
internal  
private  
within a grouping, place the view and pure functions last

## \[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.