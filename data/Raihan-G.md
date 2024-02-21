# GAS OPTIMIZATION

# SUMMARY
|      |  issue  |  instance  |
|------|---------|------------|
|[G‑01]|Most important: avoid zero to one storage writes where possible|10|
|[G‑02]|Using mappings instead of arrays to avoid length checks|2|
|[G‑03]|Use storage pointers instead of memory where appropriate|1|
|[G‑04]|Use assembly to perform efficient back-to-back calls|5|
|[G‑05]|Avoid emitting storage values|3|
|[G‑06]|Structs can be modified to fit in fewer storage slots|1|
|[G‑07]|Pre-increment and pre-decrement are cheaper than +1 ,-1|11|
|[G‑08]|Gas Optimization: Favor Ternary Operation Over If-Else Statement|2|
|[G‑09]|Consider using OZ EnumerateSet in place of nested mappings|8|
|[G‑10]|Don’t make variables public unless it is necessary to do so|2|
|[G‑11]|It is sometimes cheaper to cache calldata|1|
|[G‑12]|Gas Optimization: Use Assembly for Writing Address Storage Values|9|
|[G‑13]|Sort Solidity operations using short-circuit mode|1|
|[G‑14]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|3|
|[G‑15]|Do not calculate constants|3|
|[G‑16]|Avoid having ERC20 token balances go to zero, always keep a small amount|3|
|[G‑17]|ERC1155 is a cheaper non-fungible token than ERC721|2|
|[G‑18]|Use one ERC1155 or ERC6909 token instead of several ERC20 tokens|1|
|[G‑19]|Consider using alternatives to OpenZeppelin|7|
|[G‑20]|Using assembly to revert with an error message|17|
|[G‑21]|Common math operations, like min and max have gas efficient alternatives|1|

## [G-01] Most important: avoid zero to one storage writes where possible
Initializing a storage variable is one of the most expensive operations a contract can do.
When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).
This is why the Openzeppelin reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1. It only costs 5,000 gas to alter a storage variable from non-zero to non-zero.


There are 10 instances of this issue:
```solidity
File:  src/GameItems.sol
64   uint256 _itemCount = 0;   
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L64


```solidity
File:  src/MergingPool.sol
29  uint256 public roundId = 0;

32  uint256 public totalPoints = 0;  
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L29


```solidity
File:  src/GameItems.sol
64  uint256 _itemCount = 0;    
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L64


```solidity
File:  src/MergingPool.sol
29  uint256 public roundId = 0;

32  uint256 public totalPoints = 0;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L29


```solidity
File:  src/RankedBattle.sol
56  uint256 public totalBattles = 0;

59  uint256 public globalStakedAmount = 0;

62  uint256 public roundId = 0;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L56


```solidity
File:  src/StakeAtRisk.sol
27  uint256 public roundId = 0;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L27



## [G-02] Using mappings instead of arrays to avoid length checks
When storing a list or group of items that you wish to organize in a specific order and fetch with a fixed key/index, it’s common practice to use an array data structure. This works well, but did you know that a trick can be implemented to save 2,000+ gas on each read using a mapping?


See the example below
```solidity
/// get(0) gas cost: 4860 
contract Array {
    uint256[] a;

    constructor() {
        a.push() = 1;
        a.push() = 2;
        a.push() = 3;
    }

    function get(uint256 index) external view returns(uint256) {
        return a[index];
    }
}

/// get(0) gas cost: 2758
contract Mapping {
    mapping(uint256 => uint256) a;

    constructor() {
        a[0] = 1;
        a[1] = 2;
        a[2] = 3;
    }

    function get(uint256 index) external view returns(uint256) {
        return a[index];
    }
}
```
Just by using a mapping, we get a gas saving of 2102 gas. Why? Under the hood when you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index (i.e an index strictly less than the length of the array) else it reverts with a panic error (Panic(0x32) to be precise). This prevents from reading unallocated or worse, allocated storage/memory locations.
Due to the way mappings are (simply a key => value pair), no check like that exists and we are able to read from the a storage slot directly. It’s important to note that when using mappings in this manner, your code should ensure that you are not reading an out of bound index of your canonical array.


There are 2 instance(s) of this issue:
```solidity
File:  src/FighterFarm.sol
69  FighterOps.Fighter[] public fighters;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L69


```solidity
File:  src/GameItems.sol
55  GameItemAttributes[] public allGameItemAttributes;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L55

## [G-03] Use storage pointers instead of memory where appropriate
In Solidity, storage pointers are variables that reference a location in storage of a contract. They are not exactly the same as pointers in languages like C/C++.
It is helpful to know how to use storage pointers efficiently to avoid unnecessary storage reads and perform gas-efficient storage updates.
Here’s an example showing where storage pointers can be helpful.
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract StoragePointerUnOptimized {
    struct User {
        uint256 id;
        string name;
        uint256 lastSeen;
    }

    constructor() {
        users[0] = User(0, "John Doe", block.timestamp);
    }

    mapping(uint256 => User) public users;

    function returnLastSeenSecondsAgo(uint256 _id) public view returns (uint256) {
        User memory _user = users[_id];
        uint256 lastSeen = block.timestamp - _user.lastSeen;
        return lastSeen;
    }
}
```
Above, we have a function that returns the last seen of a user at a given index. It gets the lastSeen value and subtracts that from the current block.timestamp. Then we copy the whole struct into memory and get the lastSeen which we use in calculating the last seen in seconds ago. This method works well but is not so efficient, this is because we are copying all of the struct from storage into memory including variables we don’t need. Only if there was a way to only read from the lastSeen storage slot (without assembly). That’s where storage pointers come in.
```
// This results in approximately 5,000 gas savings compared to the previous version.
contract StoragePointerOptimized {
    struct User {
        uint256 id;
        string name;
        uint256 lastSeen;
    }

    constructor() {
        users[0] = User(0, "John Doe", block.timestamp);
    }

    mapping(uint256 => User) public users;

    function returnLastSeenSecondsAgoOptimized(uint256 _id) public view returns (uint256) {
        User storage _user = users[_id]; 
        uint256 lastSeen = block.timestamp - _user.lastSeen;
        return lastSeen;
    }
}
```
“The above implementation results in approximately 5,000 gas savings compared to the first version”. Why so, the only change here was changing memory to storage and we were told that anything storage is expensive and should be avoided?
Here we store the storage pointer for users[_id] in a fixed sized variable on the stack (the pointer of a struct is basically the storage slot of the start of the struct, in this case, this will be the storage slot of user[_id].id). Since storage pointers are lazy (meaning they only act(read or write) when called or referenced). Next we only access the lastSeen key of the struct. This way we make a single storage load then store it on the stack, instead of 3 or possibly more storage loads and a memory store before taking a small chunk from memory unto the stack.

Note: When using storage pointers, it’s important to be careful not to reference dangling pointers. (Here is a video tutorial on dangling pointers by one of RareSkills' instructors).

There are 1 instance(s) of this issue:
```solidity
File:  src/GameItems.sol
257  string memory customURI = _tokenURIs[tokenId];
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L257

## [G-04] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)


There are 5 instance(s) of this issue:
```solidity
File:  src/FighterFarm.sol
250             require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
251            _mintpassInstance.burn(mintpassIdsToBurn[i]);


375    _neuronInstance.approveSpender(msg.sender, rerollCost);
376    bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L250-L251


```solidity
File:  src/GameItems.sol
163        _neuronInstance.approveSpender(msg.sender, price);
164        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, price);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L163-L164


```solidity
File:  src/RankedBattle.sol
250        _neuronInstance.approveStaker(msg.sender, address(this), amount);
251        bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L250-L251


```solidity
File:  src/VoltageManager.sol
95        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
96        _gameItemsContractInstance.burn(msg.sender, 0, 1);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L95-L96


## [G-05] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.


There are 3 instance(s) of this issue:

```solidity
File:  src/GameItems.sol
// @audit `_itemCount` is stroge variable
231   emit Locked(_itemCount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L231


```solidity
File:  src/VoltageManager.sol
// @audit `ownerVoltage[msg.sender]` is stroge access
98   emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);

// @audit `ownerVoltage[spender]` is stroge access
111 emit VoltageRemaining(spender, ownerVoltage[spender]);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L98



## [G-06] Structs can be modified to fit in fewer storage slots
Some member types can be safely modified, and as result, these struct will fit in fewer storage slots. Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There are 1 instance(s) of this issue:
```solidity
File:  src/FighterOps.sol
    struct FighterPhysicalAttributes {
        uint256 head;
        uint256 eyes;
        uint256 mouth;
        uint256 body;
        uint256 hands;
        uint256 feet;
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L26-L33


## [G-07] Pre-increment and pre-decrement are cheaper than +1 ,-1

There are 11 instance(s) of this issue:
```solidity
File:  src/GameItems.sol
234   _itemCount += 1;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L234


```solidity
File:  src/MergingPool.sol
131  roundId += 1;

160  claimIndex += 1;

180  numRewards += 1;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L131


```solidity
File:  src/RankedBattle.sol
236  roundId += 1;

348   totalBattles += 1;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L236

```solidity
File:  src/GameItems.sol
131        generation[fighterType] += 1;
132        maxRerollsAllowed[fighterType] += 1;

293        numTrained[tokenId] += 1;
294        totalNumTrained += 1;

378        numRerolls[tokenId] += 1;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L131-L132

## [G-08] Gas Optimization: Favor Ternary Operation Over If-Else Statement

**Explanation of Issue:**

Using a ternary operation instead of an if-else statement can result in reduced gas costs, making it a more gas-efficient choice for conditional assignments.

**Examples of Code:**

  - *Non-optimized Code:*
    ```solidity
    contract NonOptimizedContract {
        function conditionallyAssign(uint256 value) public pure returns (uint256) {
            // Non-optimized: Using if-else statement for conditional assignment
            if (value > 0) {
                return value;
            } else {
                return 0;
            }
        }
    }
    ```

  - *Optimized Code:*
    ```solidity
    contract OptimizedContract {
        function conditionallyAssign(uint256 value) public pure returns (uint256) {
            // Optimized: Using ternary operation for conditional assignment
            return (value > 0) ? value : 0;
        }
    }
    ```

**Reason:**  

The non-optimized code utilizes an if-else statement for a simple conditional assignment. While this approach is readable, it involves additional opcodes, potentially leading to higher gas costs.

In the optimized code, a ternary operation is used for the same conditional assignment. Ternary operations generally result in fewer opcodes and are therefore more gas-efficient. This optimization is particularly valuable in scenarios where gas-conscious development is a priority, and the conditionals involve simple assignments.

There are 2 instance(s) of this issue:
```solidity
File:  src/GameItems.sol
        if (stakingStatus) {
            emit Locked(tokenId);
        } else {
            emit Unlocked(tokenId);
        }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L271-L275

```solidity
File:  src/GameItems.sol
        if (transferable) {
          emit Unlocked(tokenId);
        } else {
          emit Locked(tokenId);
        }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L129-L133


## [G-09] Consider using OZ EnumerateSet in place of nested mappings             
Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).
A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.
OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.

There are 8 instance(s) of this issue:

```solidity 
File:  src/AiArenaHelper.sol
31  mapping(address => mapping(uint8 => uint8)) public passesClaimed;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L31


```solidity
File:  src/FighterFarm.sol
88   mapping(address => mapping(uint8 => uint8)) public nftsClaimed;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L88

```solidity
File:  src/GameItems.sol
74  mapping(address => mapping(uint256 => uint256)) public allowanceRemaining;

77  mapping(address => mapping(uint256 => uint256)) public dailyAllowanceReplenishTime;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L74


```solidity
File:  src/RankedBattle.sol
113   mapping(address => mapping(uint256 => uint256)) public accumulatedPointsPerAddress;

116   mapping(uint256 => mapping(uint256 => uint256)) public accumulatedPointsPerFighter;

125   mapping(uint256 => mapping(uint256 => bool)) public hasUnstaked;

134   mapping(uint256 => mapping(uint256 => bool)) _calculatedStakingFactor;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L113


```solidity
File:  src/StakeAtRisk.sol
46  mapping(uint256 => mapping(uint256 => uint256)) public stakeAtRisk;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L46


## [G-10] Don’t make variables public unless it is necessary to do so
A public storage variable has an implicit public function of the same name. A public function increases the size of the jump table and adds bytecode to read the variable in question. That makes the contract larger.

Remember, private variables aren’t private, it’s not difficult to extract the variable value using web3.js.

This is especially true for constants which are meant to be read by humans rather than smart contracts.

There are 2 instance(s) of this issue:
```solidity
File:  src/StakeAtRisk.sol
46  mapping(uint256 => mapping(uint256 => uint256)) public stakeAtRisk;

// @audit do not need public becuse it have geter function
    function getStakeAtRisk(uint256 fighterId) external view returns(uint256) {
        return stakeAtRisk[roundId][fighterId];
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L46

```solidity
File:  src/RankedBattle.sol
122  mapping(uint256 => uint256) public rankedNrnDistribution;

// @audit do not need public becuse it have geter function
    function getNrnDistribution(uint256 roundId_) public view returns(uint256) {
        return rankedNrnDistribution[roundId_];
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L122


## [G-11] It is sometimes cheaper to cache calldata
Although the calldataload instruction is a cheap opcode, the solidity compiler will sometimes output cheaper code if you cache calldataload. This will not always be the case, so you should test both possibilities.
```
contract LoopSum {
    function sumArr(uint256[] calldata arr) public pure returns (uint256 sum) {
        uint256 len = arr.length;
        for (uint256 i = 0; i < len; ) {
            sum += arr[i];
            unchecked {
                ++i;
            }
        }
    }
}
```
There are 1 instance(s) of this issue:
```solidity
File: src/FighterFarm.sol
    function redeemMintPass(
        uint256[] calldata mintpassIdsToBurn, // Gas
        uint8[] calldata fighterTypes,
        uint8[] calldata iconsTypes,
        string[] calldata mintPassDnas,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
            modelHashes.length == modelTypes.length
        );
        for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L233-L249


## [G-12] Gas Optimization: Use Assembly for Writing Address Storage Values

**Explanation of Issue:**
When writing to address storage values in Solidity, using assembly can provide a gas-efficient alternative. Assembly allows for direct interaction with the Ethereum Virtual Machine (EVM) and enables low-level operations that are not achievable in Solidity. By using assembly to write to address storage values, you can potentially lower the gas cost associated with storage operations.

**Examples of Code:**

- *Non-optimized Code:*
```solidity
contract MyContract {
    address private myAddress;
    
    function setAddressWithoutAssembly(address newAddress) public {
        myAddress = newAddress;
    }
}
```

- *Optimized Code:*
```solidity
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress);
        }
    }
}
```

**Reason:**  
In the non-optimized code, the address value is assigned directly in Solidity. In the optimized code, assembly is used to write to the storage location. This can result in a potential gas savings as assembly provides direct access to the Ethereum Virtual Machine (EVM) and allows for more efficient storage operations. However, it's essential to use assembly judiciously and consider the trade-offs in terms of readability and security when employing low-level operations. This optimization is particularly beneficial in scenarios where minimizing gas costs is a critical concern.

There are 9 instance(s) of this issue:
```solidity
File:  src/AiArenaHelper.sol
47  delegatedAddress = _delegatedAddress;

48  founderAddress = _founderAddress;

58  founderAddress = _newFounderAddress;

81  fighterFarmContractAddress = _fighterFarmAddress;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L47


```solidity
File:  src/FighterFarm.sol
107  _ownerAddress = ownerAddress;

108  _delegatedAddress = delegatedAddress;

109  treasuryAddress = treasuryAddress_;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L107


```solidity
File:  src/FighterFarm.sol
173  _mergingPoolAddress = mergingPoolAddress;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L137


```solidity
File:  src/GameItems.sol
110   _ownerAddress = newOwnerAddress;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L110


## [G-13] Sort Solidity operations using short-circuit mode

**Explanation of Issue:**
The issue involves optimizing the sequencing of Solidity operations using short-circuit mode. Short-circuiting utilizes OR/AND logic to arrange operations based on their gas costs. By placing low gas cost operations at the front and high gas cost operations at the back, subsequent high-cost Ethereum virtual machine operations can be skipped (short-circuited) if the initial low-cost operation evaluates to true.

**Examples of Code:**
- f(x) is a low gas cost operation 
- g(y) is a high gas cost operation 
- *Non-optimized Code:*
```solidity
// Operations not sorted by gas cost
function performOperations(uint256 x, uint256 y) public {
    // High gas cost operation
    if (g(y) || f(x)) {
        // Code to execute if the condition is true
    }
}
```

- *Optimized Code:*
```solidity
// Operations sorted by gas cost using short-circuit mode
function performOperations(uint256 x, uint256 y) public {
    // Low gas cost operation first
    if (f(x) || g(y)) {
        // Code to execute if the condition is true
    }
}
```

**Reason:**
The reason for this optimization is to improve gas efficiency by leveraging short-circuiting. By sorting operations based on their gas costs, with low-cost operations placed before high-cost operations, subsequent high-cost Ethereum virtual machine operations can be skipped if the preceding low-cost operation evaluates to true. In the non-optimized code example, the `g(y)` operation is executed first, and if it returns true (or a non-zero value), the `f(x)` operation is not executed due to the short-circuit behavior of the `||` operator. However, this does not follow the intended gas optimization technique. In the optimized code example, the `f(x)` operation is executed first, and if it returns true, the `g(y)` operation is not executed. This optimization reduces unnecessary gas consumption and results in cost savings during contract execution.

There are 1 instance(s) of this issue:

`fighterFarmContractAddress` is storage variable, access of storage use more gas then `ownerOf` function call.
```solidity
File:  src/AiArenaHelper.sol
138  require(msg.sender == fighterFarmContractAddress || msg.sender == ownerOf(_tokenId));
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L138

Fixed code
```diff
- require(msg.sender == fighterFarmContractAddress || msg.sender == ownerOf(_tokenId));
+ require(msg.sender == ownerOf(_tokenId) || msg.sender == fighterFarmContractAddress);
```


## [G-14] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

**Explanation of Issue:**
When using constant variables in Solidity, expressions that involve expensive operations like a call to `keccak256()` are evaluated at runtime and their value is included in the bytecode of the contract. This means that every time the contract is deployed, the expensive operations in the constant expression will be executed, even if the result is always the same. As a result, this can lead to higher gas costs.

**Examples of Code:**
- *Non-optimized Code:*
```solidity
bytes32 constant MY_HASH = keccak256("my string");
```
- *Optimized Code:*
```solidity
bytes32 immutable MY_HASH = keccak256("my string");
```

**Reason:**  
By using an immutable variable instead of a constant, we can avoid executing the expensive operation `keccak256()` every time the contract is deployed. The immutable variable is evaluated at compilation time, and its value is included directly in the bytecode. This optimization reduces the gas costs associated with deploying the contract.


There are 3 instances of this issue:
```solidity
File:  src/Neuron.sol
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    /// @notice Role for spending tokens.
    bytes32 public constant SPENDER_ROLE = keccak256("SPENDER_ROLE");
    
    /// @notice Role for staking tokens.
    bytes32 public constant STAKER_ROLE = keccak256("STAKER_ROLE");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L28-L34

## [G-15] Do not calculate constants

**Explanation of Issue:**
Calculating constants in Solidity can lead to inefficient gas usage. Constant variables are replaced at compile-time, which means that expressions assigned to constant variables are recomputed each time the variable is used. This results in unnecessary gas consumption.

**Examples of Code:**
  - *Non-optimized Code:*
    ```solidity
    uint256 constant FEE_PERCENTAGE = (100 * 100) / totalSupply;
    ```
  - *Optimized Code:*
    ```solidity
    uint256 constant FEE_PERCENTAGE = 1;
    ```

**Reason:**  
Calculating constants each time they are used wastes gas due to redundant computations. By assigning a constant value directly, we avoid recomputing the expression, leading to more efficient gas usage and better contract performance.
[Reffrence](https://code4rena.com/reports/2022-07-golom#g-24-do-not-calculate-constants)

There are 3 instances of this issue:
```solidity
File:  src/Neuron.sol
    uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;

    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
    uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;

    /// @notice Maximum supply of NRN tokens.
    uint256 public constant MAX_SUPPLY = 10**18 * 10**9;
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L37-L43

## [G-16] Avoid having ERC20 token balances go to zero, always keep a small amount
This is related to the avoiding zero writes section above, but it’s worth calling out separately because the implementation is a bit subtle.
If an address is frequently emptying (and reloading) it’s account balance, this will lead to a lot of zero to one writes.

There are 3 instance(s) of this issue:
```solidity
File:  src/GameItems.sol
244  _burn(account, tokenId, amount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L244


```solidity
File:  src/Neuron.sol
164  _burn(msg.sender, amount);

202  _burn(account, amount);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L164


## [G-17] ERC1155 is a cheaper non-fungible token than ERC721
The ERC721 balanceOf function is rarely used in practice but adds a storage overhead whenever a mint and transfer happens. ERC1155 tracks balance per id, and also uses the same balance to track ownership of the id. If the maximum supply for each id is one, then the token becomes non-fungible per each id.

There are 2 instance(s) of this issue:
```solidity
File:  src/AiArenaHelper.sol
4 import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L4


```solidity
File:  src/FighterFarm.sol
9  import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L9


## [G-18] Use one ERC1155 or ERC6909 token instead of several ERC20 tokens
This was the original intent of the ERC1155 token. Each individual token behaves like and ERC20, but only one contract needs to be deployed.
The drawback of this approach is that the tokens will not be compatible with most DeFi swapping primitives.
ERC1155 uses callbacks on all of the transfer methods. If this is not desired, [ERC6909](https://eips.ethereum.org/EIPS/eip-6909) can be used instead.

There are 1 instance(s) of this issue:
```solidity
File:  src/Neuron.sol
4  import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L4

## [G-19] Consider using alternatives to OpenZeppelin
OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.
Two examples of such alternatives are Solmate and Solady.
Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.

There are 7 instance(s) of this issue:
```solidity
File: src/AiArenaHelper.sol
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L4-L5


```solidity
File:  src/FighterFarm.sol
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L9-L10


```solidity
File:  src/GameItems.sol
import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L5


```solidity
File:  src/Neuron.sol
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L4-L5




## [G-20] Using assembly to revert with an error message
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.


Here’s an example;
```
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```
From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

There are 17 instances of this issue:

```solidity
File: src/AiArenaHelper.sol
133       require(probabilities.length == 6, "Invalid number of attribute arrays");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol

```solidity
File: src/FighterFarm.sol
373       require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol

```solidity
File: src/GameItems.sol
150         require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol

```solidity
File: src/MergingPool.sol
120:         require(winners.length == winnersPerPeriod, "Incorrect number of winners");

121:         require(!isSelectionComplete[roundId], "Winners are already selected");

196:         require(msg.sender == _rankedBattleAddress, "Not Ranked Battle contract address");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol

```solidity
File: src/Neuron.sol
156:         require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");

157:         require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol

```solidity
File: src/RankedBattle.sol
245:         require(amount > 0, "Amount cannot be 0");

246:         require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");

247:         require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance");

248:         require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");

271:         require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");

295:         require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol

```solidity
File: src/StakeAtRisk.sol
79:         require(msg.sender == _rankedBattleAddress, "Not authorized to set new round");

94:         require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");

122:         require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol

## [G-21] Common math operations, like min and max have gas efficient alternatives
Unoptimized
```
function max(uint256 x, uint256 y) public pure returns (uint256 z) {
	z = x > y ? x : y;
}
```
Optimized
```
function max(uint256 x, uint256 y) public pure returns (uint256 z) {
    /// @solidity memory-safe-assembly
    assembly {
        z := xor(x, mul(xor(x, y), gt(y, x)))
    }
}
```
The code above is taken from the math section of the Solady Library, more math operations can be found there. It is worth exploring the library to see what gas efficient operations are available to you.


The reason the above example is more gas efficient is because the ternary operator (and in general, code with conditionals in it) contain conditional jumps in the opcodes, which are more costly.

There are 1 instance(s) of this issue:
```solidity
File:  src/FighterFarm.sol
472  uint256 newDna = fighterType == 0 ? dna : uint256(fighterType);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L472

