
# QA Report : AI Arena Contest | 9th Feb 2024 - 21st Feb 2024

## Executive summary
 **AI Arena** is a PvP platform fighting game where the fighters are AIs that were trained by humans.

## Overview

| **Project Name**	| **AI Arena**                                            |
| --------------------- | ------------------------------------------------------- |
| **Repository**   	| [Github](https://github.com/code-423n4/2024-02-ai-arena)|
| **Website**      	| [AI Arena](https://aiarena.io/#/)                       |
| **X(Twitter)**   	| [𝕏](https://twitter.com/aiarena_)             	  |
| **Discord**      	| [Discord](https://discord.gg/aiarena)                   |
| **Total SLoC**  	| **1197** over **8** contracts                           |

---

## Findings Description

| **ID**              | **Title**                                                                                                | **Severity**       |
| --------------- | ---------------------------------------------------------------------------------------------------- | -------------- |
| [L - 01](#l-01-unspecified-compiler-version-pragma) | Unspecified Compiler version pragma                              | _Low_          |
| [L - 02](#l-02-redundant-operation-of-initializing-attributeprobabilities-mapping-in-constructor) | Redundant operation of initializing attributeProbabilities mapping in `constructor`                | _Low_          |
| [L - 03](#l-03-use-openzeppelins-ownable-contract-for-ownership-related-actions) | Use OpenZeppelin's Ownable contract for `ownership` related actions                | _Low_          |
| [L - 04](#l-04-setupairdrop-function-arg-recipients-can-have-duplicates) | `setupAirdrop()` function arg: recipients[] can have duplicates                    | _Low_          |
| [L - 05](#l-05-unsafe-erc20-operations-use-safeerc20-library) | Unsafe ERC20 Operation(s), use `SafeERC20` library            | _Low_          |
| [L - 06](#l-06-transferfrom-returns-a-bool-value-in-claim-which-is-not-checked) | `transferFrom()` returns a bool value in `claim()`, which is not checked                  | _Low_          |
| [L - 07](#l-07-consideration-for-timestamp-overflow-in-_replenishvoltage) | Consideration for Timestamp overflow in `_replenishVoltage()`                           | _Low_          |
| [NC-01](#nc-01-contracts-doesnt-follow-the-layout-ordering) | Contracts doesn't follow the layout ordering                  | _Non Critical_ |
| [NC-02](#nc-02-functions-mutating-storage-can-better-emit-events-for-off-chain-tracking) | Functions mutating storage, can better emit events for off-chain tracking                                                     | _Non Critical_ |
| [NC-03](#nc-03-better-to-emit-events-before-making-an-external-call) | Better to emit events before making an external call                    | _Non Critical_ |

---
---

## [L-01] Unspecified Compiler version pragma

#### Description: 
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### Code Snippets:  
[Instance-1](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L2), [Instance-2](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L2), [Instance-3](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L2), [Instance-4](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L2),   [Instance-5](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L2),  [Instance-6](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L2),   [Instance-7](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L2),  [Instance-8](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L2)
#### Recommendation: 
Use a fixed pragma solidity version across all contracts for compatibility, and since v0.8.20 onwards Solidity introduced `PUSH0` opcode, which is still not supported by other chains. Thus, v0.8.19 has been recommended here. If we are not planning for multiple chains for deployment, we can go ahead with v0.8.20
```diff
- pragma solidity >=0.8.0 <0.9.0;
+ pragma solidity 0.8.19;  
```

##
## [L-02] Redundant operation of initializing attributeProbabilities mapping in `constructor`
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
#### Recommendation: 
Based on the implementation of the `addAttributeProbabilities()` it appears that, it just initializes the attributeProbabilities mapping, but in the constructor `for` loop we also initialize the attributeToDnaDivisor mapping. So `addAttributeProbabilities()` and `addAttributeDivisor()` should be called in `constructor` or either keep the `for` loop in the constructor which does the initialization for both mappings.

##

## [L-03] Use OpenZeppelin's Ownable contract for `ownership` related actions
#### Description: 
There are missing checks for owner address, though owner is *trusted* entity, but following best practices is always appreciated.
#### Code Snippets: 
[Neuron.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85-L88), [AiArenaHelper.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L61-L64) , [GameItems.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L108-L111), 
[MergingPool.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L89-L92),  
[RankedBattle.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L61-L64) ,
[FighterFarm.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L120-L123), 
[Neuron.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L85-L88)

**Code**:
```javascript		
 function transferOwnership(address newOwnerAddress) external {
     require(msg.sender == _ownerAddress);
     _ownerAddress = newOwnerAddress;
}
```
#### Recommendation: 
Inherit OZ's Ownable contract and override the `transferOwnership()` function.

##
## [L-04] `setupAirdrop()` function arg: recipients[] can have duplicates 
#### Description: 
If a recipient address appears more than once in the `recipients` array, the allowances for that recipient will be overwritten in each iteration of the loop. This means that the final allowance for a duplicate recipient will be set to the value specified in the last iteration of the loop.


#### [Neuron.sol::setupAirdrop()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L127-L134) :
**Code:**
```javascript		
function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
        require(isAdmin[msg.sender]);
        require(recipients.length == amounts.length);
        uint256 recipientsLength = recipients.length;
        for (uint32 i = 0; i < recipientsLength; i++) {
            _approve(treasuryAddress, recipients[i], amounts[i]);
        }
}
```

#### Recommendation: 
You might consider updating the logic to handle duplicate recipients more appropriately. One approach could be to ***check whether an allowance already exists for a recipient before setting a new one, and perhaps summing up the amounts for duplicate recipients instead of overwriting the allowance***.

##
## [L-05] Unsafe ERC20 Operation(s), use `SafeERC20` library
#### Description: 
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's  `SafeERC20`  library or at least to wrap each operation in a  `require`  statement.

[Neuron.sol#L171-L177](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L171-L177)
[Neuron.sol#L184-L190](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L184-L190)

**Code**:
```javascript
function approveSpender(address account, uint256 amount) public {
        require(
            hasRole(SPENDER_ROLE, msg.sender), 
            "ERC20: must have spender role to approve spending"
        );
// To circumvent ERC20's approve functions front running
// vulnerability use OpenZeppelin's SafeERC20
        _approve(account, msg.sender, amount);
    }
```
#### Recommendation:
Import SafeERC20 to use it's call on IERC20 interface: whose functions are implemented in Neuron.sol (ERC20 Token) contract.
SafeERC20 library's safe{Increase|Decrease}Allowance functions can be used .
```diff
+ import {SafeERC20} from "openzeppelin/token/utils/SafeERC20.sol";
+ using SafeERC20 for IERC20;

// ...

+ IERC20(token).safeTransferFrom(msg.sender, address(this), amount);
```


##
## [L-06] `transferFrom()` returns a bool value in `claim()`, which is not checked
#### Description: 
The implementation of `tranferFrom()` in ERC20 contract has a bool value to be returned, which is not checked while using it in our `claim()`. 
[Neuron.sol::claim()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L138-L145) 
```javascript
    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
        transferFrom(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }
```

#### Recommendation: 
Introduce a `bool` variable to check whether if `transferFrom()` returns true. Add `require()` to check and revert in case of failure.

```diff
- transferFrom(treasuryAddress, msg.sender, amount);
+ bool success = transferFrom(treasuryAddress, msg.sender, amount); 
+ require(success, "ERC20 transfer failed");
```

Alternatively, we can  mitigate this issue by using `SafeERC20` library of OpenZeppelin.
```javascript
import {SafeERC20} from "openzeppelin/token/utils/SafeERC20.sol";

// ...

IERC20(token).safeTransferFrom(msg.sender, address(this), amount);
```
##
## [L-07] Consideration for Timestamp overflow in `_replenishVoltage()`
#### Description:
In the provided function `_replenishVoltage()`, the `ownerVoltageReplenishTime` is set using a `uint32` data type for storing which can have maximum timestamp till August 2106. Using *uint32* is no better than *uint256*, on the contrary, the later is more gas efficient.

<u>*Timestamp Overflow:*</u> The Unix timestamp (block.timestamp) represents the number of seconds that have passed since January 1, 1970 (UTC). If your application is designed to last for a long time, a `uint32` may overflow sooner than a `uint64`. A `uint32` can represent timestamps until around the year 2106, while a `uint64` can cover an astronomically longer time.

[VoltageManager.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L117-#L120):
**Code**:
```javascript
function _replenishVoltage(address owner) private {
    ownerVoltage[owner] = 100;
//q for uint32 max can be aug -12 2106, better for uint64?
    ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days);
}  
```
#### Recommendation:
Update `mapping(address => uint32) public ownerVoltageReplenishTime;` to have `value` of type `uint64`.
```diff
- mapping(address => uint32) public ownerVoltageReplenishTime;
+ mapping(address => uint64) public ownerVoltageReplenishTime;
```
***
---

## [NC-01] Contracts doesn't follow the layout ordering
#### Description :
The order in which you layout contract elements in Solidity can contribute to code readability, maintainability, and overall organization. Helps the fellow engineers/ auditors understand the code with less pain.
#### Recommendation:

Inside each contract, library or interface, use the [following order](https://docs.soliditylang.org/en/latest/style-guide.html) :

1.  Type declarations
2.  State variables    
3.  Events    
4.  Errors    
5.  Modifiers
6.  Functions
##

## [NC-02] Functions mutating storage, can better emit events for off-chain tracking
#### Description:

Transfers ownership to new EOA. This update of storage can better be acknowledged by the protocol team, to reduce unaware centralization risk. 
Similarly, there are many other instances present in the contracts, linked here:

[AiArenaHelper.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L61-L64) , [GameItems.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L108-L111), 
[MergingPool.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L89-L92),  
[RankedBattle.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L61-L64) ,
[FighterFarm.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L120-L123), 
[Neuron.sol::transferOwnership](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L85-L88) , 
[StakeAtRisk.sol::setNewRound](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L78-L84)

**Code**:
```javascript		
 function transferOwnership(address newOwnerAddress) external {
     require(msg.sender == _ownerAddress);
     _ownerAddress = newOwnerAddress;
}
```
#### Recommendation:

Declare an event in the contract and emit it in desired functions.
### Another Instance:
#### [AiArenaHelper.sol::deleteAttributeProbabilities](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L144-L151)
```javascript
function deleteAttributeProbabilities(uint8 generation) public {
        require(msg.sender == _ownerAddress);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = new uint8[](0);
        }
    }
```
## 
## [NC-03] Better to emit events before making an external call
#### Description: 
It is considered best practice to follow pattern of event emission before making any external call.
In the below code snippet:
[Neuron.sol::claim()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L138-L145) 
```javascript
    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
        transferFrom(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }
```

#### Recommendation: 
Emit the event before making `transferFrom()` external call given in the code snippet below.

```javascript
    function claim(uint256 amount) external { 
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        ); 
        emit TokensClaimed(msg.sender, amount); 
        transferFrom(treasuryAddress, msg.sender, amount);
    }
```
***
---
