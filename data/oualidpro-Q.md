## Summary
## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Unspecific compiler version pragma (New Instances) | 1 |
| [L-2](#L-2) | Forging new fighter types | 1 |
| [L-3](#L-3) | Possible signature replay | 1 |
| [L-4](#L-4) | `_setupRole` function is deprecated | 3 |

Total: 6 instances over 4 issues


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | [NatSpec] Natspec `@dev` is missing from documentation comments (New Instances) | 7 |
| [NC-2](#NC-2) | [NatSpec] Natspec `@param` is missing (New Instances) | 5 |
| [NC-3](#NC-3) | Complex functions should include comments (New Instances) | 1 |
| [NC-4](#NC-4) | Contract does not follow the Solidity style guide's suggested layout ordering (New Instances) | 1 |
| [NC-5](#NC-5) | Control structures do not follow the Solidity Style Guide (New Instances) | 2 |
| [NC-6](#NC-6) | Consider adding emergency-stop functionality (New Instances) | 1 |
| [NC-7](#NC-7) | Events are missing sender information (New Instances) | 1 |
| [NC-8](#NC-8) | Use named return values (New Instances) | 2 |
| [NC-10](#NC-9) | Missing zero address check in functions with address parameters (New Instances) | 1 |
| [NC-11](#NC-10) | `public` functions not called by the contract should be declared `external` instead (New Instances) | 2 |
| [NC-12](#NC-11) | NatSpec `@return` argument is missing (New Instances) | 1 |
| [NC-13](#NC-12) | Uncommented fields in a struct (New Instances) | 2 |
| [NC-14](#NC-13) | Event is missing `indexed` fields (New Instances) | 1 |
| [NC-15](#NC-14) | Missing upgradability functionality (New Instances) | 1 |
| [NC-16](#NC-15) | Use the latest Solidity version for better security (New Instances) | 1 |
| [NC-17](#NC-16) | Consider using SMTChecker (New Instances) | 1 |
| [NC-18](#NC-17) | Returning a struct instead of returning many variables is better (New Instances) | 1 |

Total: 31 instances over 17 issues



## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Unspecific compiler version pragma (New Instances) | 1 |
| [L-2](#L-2) | Forging new fighter types | 1 |
| [L-3](#L-3) | Possible signature replay | 1 |
| [L-4](#L-4) | `_setupRole` function is deprecated | 3 |
### <a name="L-1"></a>[L-1] Unspecific compiler version pragma

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

2: pragma solidity >=0.8.0 <0.9.0;

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L2)

</details>

### <a name="L-2"></a>[L-2] Forging new fighter types
According to the documentation the figther type should either be 0 or 1. However, no check is performed in this function to verify if the specified value is 0 or 1.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterFarm.sol

function incrementGeneration(uint8 fighterType) external returns (uint8) {
    require(msg.sender == _ownerAddress);
    generation[fighterType] += 1; //@audit The value of fighterType is not checked
    maxRerollsAllowed[fighterType] += 1;
    return generation[fighterType];
}

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L129-L134)

</details>

### <a name="L-3"></a>[L-3] Possible signature replay
If the protocole is deployed in another blockchain, a user would be able to replay the same signature to claim fighters on the second blockchain. This is possible because the chain id is not verified.


*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterFarm.sol

    function claimFighters(
        uint8[2] calldata numToMint,
        bytes calldata signature,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        bytes32 msgHash = bytes32(keccak256(abi.encode(
            msg.sender, 
            numToMint[0], 
            numToMint[1],
            nftsClaimed[msg.sender][0],
            nftsClaimed[msg.sender][1]
        )));
        require(Verification.verify(msgHash, signature, _delegatedAddress));//@audit Possible signature replay.
        uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
        require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
        nftsClaimed[msg.sender][0] += numToMint[0];
        nftsClaimed[msg.sender][1] += numToMint[1];
        for (uint16 i = 0; i < totalToMint; i++) {
            _createNewFighter(
                msg.sender, 
                uint256(keccak256(abi.encode(msg.sender, fighters.length))),
                modelHashes[i], 
                modelTypes[i],
                i < numToMint[0] ? 0 : 1,
                0,
                [uint256(100), uint256(100)]
            );
        }
    }

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L206)

</details>


### <a name="L-4"></a>[L-4] `_setupRole` function is deprecated
For more details check the following [link](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl-_setupRole-bytes32-address-)

*Instances (3)*:
<details><summary> see instances </summary>

```solidity
File: src/Neuron.sol

function addMinter(address newMinterAddress) external {
    require(msg.sender == _ownerAddress);
    _setupRole(MINTER_ROLE, newMinterAddress);
}

/// @notice Adds a new address to the staker role.
/// @dev Only the owner address is authorized to call this function.
/// @param newStakerAddress The address to be added as a staker
function addStaker(address newStakerAddress) external {
    require(msg.sender == _ownerAddress);
    _setupRole(STAKER_ROLE, newStakerAddress);
}

/// @notice Adds a new address to the spender role.
/// @dev Only the owner address is authorized to call this function.
/// @param newSpenderAddress The address to be added as a spender
function addSpender(address newSpenderAddress) external {
    require(msg.sender == _ownerAddress);
    _setupRole(SPENDER_ROLE, newSpenderAddress);
}

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L93-L112)

</details>


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | [NatSpec] Natspec `@dev` is missing from documentation comments (New Instances) | 7 |
| [NC-2](#NC-2) | [NatSpec] Natspec `@param` is missing (New Instances) | 5 |
| [NC-3](#NC-3) | Complex functions should include comments (New Instances) | 1 |
| [NC-4](#NC-4) | Contract does not follow the Solidity style guide's suggested layout ordering (New Instances) | 1 |
| [NC-5](#NC-5) | Control structures do not follow the Solidity Style Guide (New Instances) | 2 |
| [NC-6](#NC-6) | Consider adding emergency-stop functionality (New Instances) | 1 |
| [NC-7](#NC-7) | Events are missing sender information (New Instances) | 1 |
| [NC-8](#NC-8) | Use named return values (New Instances) | 2 |
| [NC-10](#NC-9) | Missing zero address check in functions with address parameters (New Instances) | 1 |
| [NC-11](#NC-10) | `public` functions not called by the contract should be declared `external` instead (New Instances) | 2 |
| [NC-12](#NC-11) | NatSpec `@return` argument is missing (New Instances) | 1 |
| [NC-13](#NC-12) | Uncommented fields in a struct (New Instances) | 2 |
| [NC-14](#NC-13) | Event is missing `indexed` fields (New Instances) | 1 |
| [NC-15](#NC-14) | Missing upgradability functionality (New Instances) | 1 |
| [NC-16](#NC-15) | Use the latest Solidity version for better security (New Instances) | 1 |
| [NC-17](#NC-16) | Consider using SMTChecker (New Instances) | 1 |
| [NC-18](#NC-17) | Returning a struct instead of returning many variables is better (New Instances) | 1 |
### <a name="NC-1"></a>[NC-1] [NatSpec] Natspec `@dev` is missing from documentation comments (New Instances) 

*Instances (7)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

4: /// @title FighterOps library for managing fighters in the AI Arena game.
   /// @author ArenaX Labs Inc.
   /// @notice This library is used for creating and managing Fighter NFTs in the AI Arena game.
   library FighterOps {

13:     /// @notice Emitted when a new Fighter NFT is created.

25:     /// @notice Struct that defines a Fighter's physical attributes.

35:     /// @notice Struct that defines a Fighter NFT.

52:     /// @notice Emits a FighterCreated event.

64:     /// @notice Extracting the fighter attributes from the struct
        /// @param self Fighter struct
        /// @return Array of Fighter Attributes 

78:     /// @notice Gets all of the relevant fighter information 

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L78-L78)

</details>

### <a name="NC-2"></a>[NC-2] [NatSpec] Natspec `@param` is missing (New Instances) 

*Instances (5)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

13:     /// @notice Emitted when a new Fighter NFT is created.

25:     /// @notice Struct that defines a Fighter's physical attributes.

35:     /// @notice Struct that defines a Fighter NFT.

52:     /// @notice Emits a FighterCreated event.

78:     /// @notice Gets all of the relevant fighter information 

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L78-L78)

</details>

### <a name="NC-3"></a>[NC-3] Complex functions should include comments (New Instances) 
Large and/or complex functions should include comments to make them easier to understand and reduce margin for error.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

79:     function viewFighterInfo(

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79)

</details>

### <a name="NC-4"></a>[NC-4] Contract does not follow the Solidity style guide's suggested layout ordering (New Instances)
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be 1) Type declarations, 2) State variables, 3) Events, 4) Modifiers, and 5) Functions, but the contract(s) below do not follow this ordering

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

//@audit the structure definition is misplaced
26:     struct FighterPhysicalAttributes {

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L26)

</details>

### <a name="NC-5"></a>[NC-5] Control structures do not follow the Solidity Style Guide (New Instances)
See the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) section of the Solidity Style Guide

*Instances (2)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

53:     function fighterCreatedEmitter(

79:     function viewFighterInfo(

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79)

</details>

### <a name="NC-6"></a>[NC-6] Consider adding emergency-stop functionality (New Instances)
In the event of a security breach or any unforeseen emergency, swiftly suspending all protocol operations becomes crucial. Having a mechanism in place to halt all functions collectively, instead of pausing individual contracts separately, substantially enhances the efficiency of mitigating ongoing attacks or vulnerabilities. This not only quickens the response time to potential threats but also reduces operational stress during these critical periods. Therefore, consider integrating a 'circuit breaker' or 'emergency stop' function into the smart contract system architecture. Such a feature would provide the capability to suspend the entire protocol instantly, which could prove invaluable during a time-sensitive crisis management situation.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

7: library FighterOps {

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L7)

</details>

### <a name="NC-7"></a>[NC-7] Events are missing sender information (New Instances)
When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the `msg.sender` the events of these types of action will make events much more useful to end users, especially when `msg.sender` is not `tx.origin`.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

61:         emit FighterCreated(id, weight, element, generation);

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L61-L61)

</details>

### <a name="NC-8"></a>[NC-8] Use named return values (New Instances)
Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas. The cases below are where there currently is at most one return statement, which is ideal for named returns.

*Instances (2)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

67:     function getFighterAttributes(Fighter storage self) public view returns (uint256[6] memory) {

79:     function viewFighterInfo(
            Fighter storage self, 
            address owner
        ) 
            public 
            view 
            returns (
                address,

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79-L86)

</details>

### <a name="NC-9"></a>[NC-9] Missing zero address check in functions with address parameters (New Instances)
Adding a zero address check for each address type parameter can prevent errors.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

//@audit owner,  are not checked
79:     function viewFighterInfo(
            Fighter storage self, 
            address owner
        ) 

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79-L82)

</details>

### <a name="NC-10"></a>[NC-10] `public` functions not called by the contract should be declared `external` instead (New Instances)
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*Instances (2)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

53:     function fighterCreatedEmitter(

79:     function viewFighterInfo(

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79)

</details>

### <a name="NC-11"></a>[NC-11] NatSpec `@return` argument is missing (New Instances)

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

// @audit the @return is missing
@notice Gets all of the relevant fighter information 
79:     function viewFighterInfo(

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79-L1)

</details>

### <a name="NC-12"></a>[NC-12] Uncommented fields in a struct (New Instances)
Consider adding comments for all the fields in a struct to improve the readability of the codebase.

*Instances (2)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

//@audit Add explanational comments to the following items `head`, `eyes`, `mouth`, `body`, `hands`, `feet`, 
26:     struct FighterPhysicalAttributes {
            uint256 head;
            uint256 eyes;
            uint256 mouth;
            uint256 body;
            uint256 hands;
            uint256 feet;
        }

//@audit Add explanational comments to the following items `weight`, `element`, `physicalAttributes`, `id`, `modelHash`, `modelType`, `generation`, `iconsType`, `dendroidBool`, 
36:     struct Fighter {
            uint256 weight;
            uint256 element;
            FighterPhysicalAttributes physicalAttributes;
            uint256 id;
            string modelHash;
            string modelType;
            uint8 generation;
            uint8 iconsType;
            bool dendroidBool;
        }

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L36-L46)

</details>

### <a name="NC-13"></a>[NC-13] Event is missing `indexed` fields (New Instances)
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

14:     event FighterCreated(

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L14)

</details>

### <a name="NC-14"></a>[NC-14] Missing upgradability functionality (New Instances)
At the begining of a project, there is always the need to modify of add something to the source code especialy if any vulnerability is discovered. Therefore, having such system is crusial at least at the first stages of the project

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

1: // SPDX-License-Identifier: MIT

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L1)

</details>

### <a name="NC-15"></a>[NC-15] Use the latest Solidity version for better security (New Instances)
Using the latest solidity version will help avoid old compiler related vulnerabilities

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

2: pragma solidity >=0.8.0 <0.9.0;

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L2)

</details>

### <a name="NC-16"></a>[NC-16] Consider using SMTChecker (New Instances)
The SMTChecker is a valuable tool for Solidity developers as it helps detect potential vulnerabilities and logical errors in the contract's code. By utilizing Satisfiability Modulo Theories (SMT) solvers, it can reason about the potential states a contract can be in, and therefore, identify conditions that could lead to undesirable behavior. This automatic formal verification can catch issues that might otherwise be missed in manual code reviews or standard testing, enhancing the overall contract's security and reliability.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

2: pragma solidity >=0.8.0 <0.9.0;

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L2-L2)

</details>

### <a name="NC-17"></a>[NC-17] Returning a struct instead of returning many variables is better (New Instances)
Returning a struct from a Solidity function instead of multiple variables offers several benefits, enhancing code clarity and efficiency. Structs allow for the grouping of related data into a single entity, making the function's return values more organized and easier to manage. This approach significantly improves readability, as it encapsulates the data logically, helping developers quickly understand the returned information's structure. Additionally, it simplifies function interfaces, reducing the potential for errors when handling multiple return values. By using structs, you can also easily extend or modify the returned data without altering the function signature, facilitating smoother updates and maintenance of your smart contract code.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

79:     function viewFighterInfo(
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
                uint16
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
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L79-L104)

</details>
