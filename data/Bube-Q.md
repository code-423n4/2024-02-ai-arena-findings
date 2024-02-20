# QA report

## Content

| Number  | Severity    | Issue Title                                                                               |
| --------| ------------|-------------------------------------------------------------------------------------------|
| [1]     | Low         | Missing two-step procedure for ownership transfer                                         |
| [2]     | Low         | Missing check for token existence in `RankedBattle::getBattleRecord` function             |
| [3]     | Low         | Missing check for token existence in `RankedBattle::getNrnDistribution` function          |
| [4]     | Low         | Incorrect `require` statement in ``Neuron::mint` function.                                |
| [5]     | Low         | Some important events are missing in `Neuron` contract                                    |
| [6]     | Low         | Unsafe `transferFrom` in `Neuron` contract                                                |
| [7]     | Low         | The `attributeProbabilities` mapping is initialized twice in the `AiArenaHelper` contract |

## [1] Low

## Title 

Missing two-step procedure for ownership transfer

## Links

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L61-L64
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L120-L123
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L108-L111
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L89-L92
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85-L88
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L167-L170
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L64-L67

## Impact
The `transferOwnership` function in smart contracts `AiArenaHelper`, `FighterFarm`, `GameItems`, `MergingPool`, `Neuron`, `RankedBattle`, `VoltageManager` doesn't check if the `newOwnerAddress` is a non-zero address. If the `newOwnerAddress` is a zero address or invalid (wrong) address, the `owner` functionality will be lost forever.

## Description
The `owner` is the authorized user in the solidity contracts: `AiArenaHelper`, `FighterFarm`, `GameItems`, `MergingPool`, `Neuron`, `RankedBattle`, `VoltageManager`. An `owner` can be updated with `transferOwnership` function in these contracts. However, the process is only completed with single transaction. If the address is updated incorrectly, the `owner` functionality will be lost forever.

The function `transferOwnership` has the same implementation in the following smart contracts: `AiArenaHelper`, `FighterFarm`, `GameItems`, `MergingPool`, `Neuron`, `RankedBattle`, `VoltageManager`.

```javascript


    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }

```

## Recommendation
Add a two-step procedure for the `transferOwnership` functionality. Also, it is a good practice to emit an event when important functions like `transferOwnership` are executed. Therefore, add an event for every successfull transfer of the ownership.

## [2] Low

## Title
Missing check for token existence in `RankedBattle::getBattleRecord` function

## Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L354-L360

## Description

The function `RankedBattle::getBattleRecord` gets the battle record for a token and returns record which is comprised of wins, ties, and losses for the `tokenId`. But if the `tokenId` is invalid, the function will return the default values: `0,0,0`. That leads to a confusion, the user will not understand that the `tokenId` is wrong, he/she will thing that these are the battle records for the `tokenId`.

```javascript

    function getBattleRecord(uint256 tokenId) external view returns(uint32, uint32, uint32) {
      return (
          fighterBattleRecord[tokenId].wins, 
          fighterBattleRecord[tokenId].ties, 
          fighterBattleRecord[tokenId].loses
      );
    }
```

## Recommendation
Add a check for existing/valid `tokenId` in the `RankedBattle::getBattleRecord` function.

## [3] Low

## Title
Missing check for token existence in `RankedBattle::getNrnDistribution` function

## Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L379-L381

## Description

The function `RankedBattle::getNrnDistribution` retrieves the nrn distribution amount for the given `roundId`. The function is `public` and anyone can call it. If a user call the function with invalid `roundId_`, the function will return `0`, because Solidity returns the default value from the type of mapping when the searched item is not existing. So the caller will not understand that the `roundId` is wrong.

```javascript

    function getNrnDistribution(uint256 roundId_) public view returns(uint256) {
        return rankedNrnDistribution[roundId_];
    }

```

## Recommendation
Add a check for existing/valid `roundId` in the `RankedBattle::getNrnDistribution` function.

## [4] Low

## Title
Incorrect `require` statement in ``Neuron::mint` function.

## Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L155-L159

## Description

The `Neuron::mint` function mints the specified amount of tokens to the given address. The function requires the sum of the `totalSupply` and the `amount` to not be more than the `MAX_SUPPLY`. But in the `requirement` statement is used only the sign for lower than `<`. That means if the `totalSupply() + amount` is equal to `MAX_SUPPLY` the function will revert and the minter will lose the possibility to mint one more token.

```javascript
    function mint(address to, uint256 amount) public virtual {
@>      require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply"); 
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }

```

But in the error message is said: `"Trying to mint more than the max supply"` which means that if the `totalSupply() + amount` is equal to `MAX_SUPPLY` the function should continue with the minting process.

## Recommendations

Add equals sign to the `require` statement:

```diff

   function mint(address to, uint256 amount) public virtual {
+       require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply");
-       require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply"); 
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }

```

## [5] Low

## Title
Some important events are missing in `Neuron` contract

## Links

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L93-L112

## Description

The `Neuron` contract includes functions `addMinter`, `addStaker`, and `addSpender` to assign respective roles. However, no events are emitted when these functions are successfully called.

```javascript

function addMinter(address newMinterAddress) external {
        require(msg.sender == _ownerAddress); //only Owner
        _setupRole(MINTER_ROLE, newMinterAddress); //add a minter role to a new address
    }

    function addStaker(address newStakerAddress) external {
        require(msg.sender == _ownerAddress);
        _setupRole(STAKER_ROLE, newStakerAddress); // add a staker role 
    }

    function addSpender(address newSpenderAddress) external {
        require(msg.sender == _ownerAddress);
        _setupRole(SPENDER_ROLE, newSpenderAddress); // add a spender role
    }

```

## Recommendations

Emit an event in the functions `addMinter`, `addStaker`, and `addSpender` when the new role is successfully set.

## [6] Low

## Title 

Unsafe `transferFrom` in `Neuron` contract

## Links

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L143

## Description

Unsafe `ERC20::transferFrom` function is used in `Neuron::claim` function. The `ERC20::transferFrom` reverts by failure. There is no check if the transfer is successfully executed.

```javascript

    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
@>      transferFrom(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }

```

## Recommendations
Check if the `transferFrom` is successfully executed or use `safeTransferFrom` function instead.

## [7] Low

## Title
The `attributeProbabilities` mapping is initialized twice in the `AiArenaHelper` contract

## Links
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L45-L51
## Description

The `attributeProbabilities` mapping is updated twice in the constructor of the `AiArenaHelper` contract. The constructor takes an array of probabilities as an argument and calls the `addAttributeProbabilities` function to initialize the probabilities for generation `0`. After calling `addAttributeProbabilities`, the constructor then enters a loop where it directly sets the `attributeProbabilities` for generation `0` again, along with setting the `attributeToDnaDivisor` for each attribute.

```javascript

    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
@>      addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
@>          attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 

```
This redundancy is unnecessary and could lead to wasted gas during contract deployment. The direct assignment in the loop is not needed since the `addAttributeProbabilities` function already performs the same action. It would be more efficient to remove the direct assignment or the function call to `addAttributeProbabilities` from the constructor to avoid duplicating the state changes.

## Recommendations
Remove the function call `addAttributeProbabilities` from the constructor to avoid duplicating the state changes.

``` diff

 constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
-       addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 

```