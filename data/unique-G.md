&nbsp;

## \[G‑01\] Save gas by preventing zero amount in mint() and burn()[](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L57C4-L72C6)

```
function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }

    /// @notice Burns the specified amount of tokens from the caller's address.
    /// @param amount The amount of tokens to be burned.
    function burn(uint256 amount) public virtual {
        _burn(msg.sender, amount);
    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L155C5-L165C6

```
_burn(account, amount);
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L202

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L93C4-L99C6

### **\[G-02\]** Compute known value `off-chain`

If You know what data to hash, there is no need to consume more computational power to hash it using keccak256 , you’ll end up consuming 2x amount of gas.

```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");


    /// @notice Role for spending tokens.
    bytes32 public constant SPENDER_ROLE = keccak256("SPENDER_ROLE");
    
    /// @notice Role for staking tokens.
    bytes32 public constant STAKER_ROLE = keccak256("STAKER_ROLE");
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L28C1-L34C68

&nbsp;

## \[G-04\] Assigning keccak operations to constant variables results in extra gas costs

&nbsp;

```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");


    /// @notice Role for spending tokens.
    bytes32 public constant SPENDER_ROLE = keccak256("SPENDER_ROLE");
    
    /// @notice Role for staking tokens.
    bytes32 public constant STAKER_ROLE = keccak256("STAKER_ROLE");
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L28C1-L34C68

&nbsp;

## \[G-05\] Duplicated require()/if() checks should be refactored to a modifier or function

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

## \[G-06\] Don’t cache value if it is only used once

If a value is only intended to be used once then it should not be cached. Caching the value will result in unnecessary stack manipulation.

```
uint32 lowerBound = numRoundsClaimed[msg.sender];
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L148

```
_ownerAddress = msg.sender;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L42

&nbsp;

&nbsp;

## \[G-07\]  `10 ** 18` can be changed to `1e18` and save some gas

`10 ** 18` can be changed to `1e18` to avoid unnecessary arithmetic operation and save gas.

```
uint256 public rerollCost = 1000 * 10**18;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L39

```
uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;

    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
    uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;

    /// @notice Maximum supply of NRN tokens.
    uint256 public constant MAX_SUPPLY = 10**18 * 10**9;
```

&nbsp;https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L37C5-L43C57

```
rankedNrnDistribution[0] = 5000 * 10**18;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L157

```
rankedNrnDistribution[roundId] = newDistribution * 10**18;
```

&nbsp;https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L220

```
(amountStaked[tokenId] + stakeAtRisk) / 10**18
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L528

## \[G-08\] Use != 0 instead of > 0

Using `> 0` uses slightly more gas than using `!= 0`. Use `!= 0` when comparing `uint` variables to zero, which cannot hold values below zero

```
if (claimIndex > 0) {
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L164

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L245

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L93C4-L99C6

## \[G-09\] It costs more gas to initialize variables to zero than to let the default of zero be applied

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L29C4-L32C38

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L56C1-L62C32

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L176

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L64

&nbsp;

## \[G-10\] Constants Variable Should Be Private for Gas Optimization

&nbsp;Using `private` for constant variables is cheaper in terms of gas usage. If the value of the constant variable is needed, it can be accessed via a getter function. In case of multiple constant variables, a single getter function could be used to return a tuple of the values of all currently-private constants.

```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    /// @notice Role for spending tokens.
    bytes32 public constant SPENDER_ROLE = keccak256("SPENDER_ROLE");
    
    /// @notice Role for staking tokens.
    bytes32 public constant STAKER_ROLE = keccak256("STAKER_ROLE");

    /// @notice Initial supply of NRN tokens to be minted to the treasury.
    uint256 public constant INITIAL_TREASURY_MINT = 10**18 * 10**8 * 2;

    /// @notice Initial supply of NRN tokens to be minted and distributed to contributors.
    uint256 public constant INITIAL_CONTRIBUTOR_MINT = 10**18 * 10**8 * 5;

    /// @notice Maximum supply of NRN tokens.
    uint256 public constant MAX_SUPPLY = 10**18 * 10**9;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L28C1-L43C57

```
uint8 public constant VOLTAGE_COST = 10;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L53

## \[G-11\] Multiple accesses of a mapping/array should use a storage pointer

Caching a mapping’s value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array.

```
require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");
        require(
            allGameItemAttributes[tokenId].finiteSupply == false || 
            (
                allGameItemAttributes[tokenId].finiteSupply == true && 
                quantity <= allGameItemAttributes[tokenId].itemsRemaining
            )
        );
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L150C8-L157C11

## \[G-12\] Caching global variables is expensive than using the variable itself

```
_ownerAddress = msg.sender;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L42

## \[G-13\] uint8 is not always cheaper than uint256

The EVM only operates on 32 bytes/ 256 bits at a time. This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.[](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L96)[](https://github.com/code-423n4/2023-10-badger/blob/main/packages/contracts/contracts/PriceFeed.sol#L612C8-L613C32)

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L20

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L53

## \[G-14\] When possible, use assembly instead of `unchecked{}`

You can also use unchecked{} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

> All contest

## [](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281)

## \[G-15\] Make fewer external calls.

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, It’s better to call one function and have it return all the data you need rather than calling a separate function for every piece of data. This might go against the best coding practices for other languages, but solidity is special.

## [](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L53C4-L84C32)

## \[G-16\] Use bytes32 rather than string/bytes.

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.[](https://github.com/code-423n4/2023-10-badger/blob/main/packages/contracts/contracts/Governor.sol#L19)

```
string name;
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L36

```
string public name = "AI Arena Game Items"; @audit

    /// @notice The symbol for this smart contract.
    string public symbol = "AGI";
```

&nbsp;https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L49

## \[G-17\] If statements that use && can be refactored into nested if statements

```
if (
                  i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
                  i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
                  i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
                ) {
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L100C17-L104C20

&nbsp;