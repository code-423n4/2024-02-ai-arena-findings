## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | `Internal` functions only called once can be inlined to save gas | 1 | - |
| [G-02] | Using fixed bytes is cheaper than using string | 5 | - |
| [G-03] | Should use arguments instead of state variable | 2 | - |
| [G-04] | Before transfer of  some functions, we should check some variables for possible gas save | 1 | - |
| [G-05] | Do not calculate constant | 3 | - |
| [G-06] | Use hardcode address instead `address(this)` | 2 | - |
| [G-07] | Pre-increment and pre-decrement are cheaper than +1 ,-1 | 2 | - |

## Gas Optimizations  

## [G-01] `Internal` functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.


```solidity
file: /src/FighterFarm.sol

447    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
        internal
        override(ERC721, ERC721Enumerable)
    {
        super._beforeTokenTransfer(from, to, tokenId);
    }


```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L447C1-L452C6


## [G-02]  Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.


```solidity
file: /src/GameItems.sol

49    string public name = "AI Arena Game Items";

52    string public symbol = "AGI";

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L49C1-L49C48


```solidity
file: /src/AiArenaHelper.sol

17    string[] public attributes = ["head", "eyes", "mouth", "body", "hands", "feet"];

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#17


```solidity
file: /src/FighterOps.sol

41        string modelHash;
42        string modelType;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L41C1-L41C26


## [G-03] Should use arguments instead of state variable 

state variables should not used in `emit`  ,  This will save near 97 gas   


```solidity
file: /src/VoltageManager.sol

///@audit ' ownerVoltage ' 

98        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);

111        emit VoltageRemaining(spender, ownerVoltage[spender]);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L98C1-L98C69


## [G-04] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything


```solidity
file: /src/StakeAtRisk.sol

143        return _neuronInstance.transfer(treasuryAddress, totalStakeAtRisk[roundId]);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol#L143C1-L143C85


## [G-05] Do not calculate constant

When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.


```solidity
file: /src/Neuron.sol

37    uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;

    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
40    uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;

    /// @notice Maximum supply of NRN tokens.
43    uint256 public constant MAX_SUPPLY = 10**18 * 10**9;

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L37C1-L45C41


## [G-06] Use hardcode address instead `address(this)`
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

```solidity
file: /src/RankedBattle.sol

250        _neuronInstance.approveStaker(msg.sender, address(this), amount);
251        bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L250C1-L251C88


## [G-07] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
file: /src/VoltageManager.sol

119        ownerVoltageReplenishTime[owner] = uint32(block.timestamp + 1 days);

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L119C1-L119C77


```solidity
file: /src/RankedBattle.sol

238        rankedNrnDistribution[roundId] = rankedNrnDistribution[roundId - 1];

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L238C1-L238C77
