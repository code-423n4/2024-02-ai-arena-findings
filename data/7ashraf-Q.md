# Quality Assurance Report

## Summary
The QA report flags important issues and offers fixes for better project performance and security. It advises against accidentally adding new fighter types and stresses the importance of checking for existing stakeholders. It also notes potential security risks like unbounded loops and missing reentrancy locks. Recommendations include avoiding hardcoded values and simplifying complex functions for clearer code. Following these suggestions will enhance the project's overall quality and security.

| Issue Number | Issue Title                                                                                      | Number of Instances |
|--------------|--------------------------------------------------------------------------------------------------|---------------------|
| L-01         | More Fighter types can be added by mistake                                                      | 1                   |
| L-02         | Check if staker already exists before adding                                                    | 1                   |
| L-03         | Possible dos for unbounded loop                                                                 | 1                   |
| L-04         | Require `amount` to be greater than zero                                                        | 1                   |
| L-05         | Add re-entrance lock                                                                            | 2                   |
| N-01         | Empty string is passed to the function, avoid hardcoded empty strings                            | 1                   |
| N-02         | Save String as a constant instead of hardcoding it in the function                               | 3                   |
| N-03         | Function should be allowed to be called only once                                                | 1                   |
| N-04         | Avoid on-chain computation                                                                      | 1                   |
| N-05         | Function is so clumped                                                                          | 1                   |

## [L-01] More Fighter types can be added by mistake 
The function can possibly add new fighter types if not handled correctly, adding un-wanted logic 
### Instances
* [FighterFarm.sol #129](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L129C1-L134C6)
```solidity
    function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
        generation[fighterType] += 1;
        maxRerollsAllowed[fighterType] += 1;
        return generation[fighterType];
    }
```
### Mitigation
* Check for fighter type either 0 or 1 before performing logic, revert if an invalid fighter type is called

## [L-02] Check if a staker already exists before adding 
### Instances
* [FighterFarm.sol #139](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L139C1-L142C6)
```solidity
    function addStaker(address newStaker) external {
        require(msg.sender == _ownerAddress);
        hasStakerRole[newStaker] = true;
    }
```
## Function Should emit an event

 
 ### Instances
* [FighterFarm.sol #139](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L139C1-L142C6)
```solidity
    function addStaker(address newStaker) external {
        require(msg.sender == _ownerAddress);
        hasStakerRole[newStaker] = true;
    }
```

## [L-03] Possible dos for unbounded loop

 ### Instances
* [FighterFarm.sol #249](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L249)
```solidity
        for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
            _mintpassInstance.burn(mintpassIdsToBurn[i]);
            _createNewFighter(
                msg.sender, 
                uint256(keccak256(abi.encode(mintPassDnas[i]))), 
                modelHashes[i], 
                modelTypes[i],
                fighterTypes[i],
                iconsTypes[i],
                [uint256(100), uint256(100)]
            );
        }
```
## [L-04] Require `amount` to be greater than zero
 ### Instances
* [RankedBattle.sol #273](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L273)
```solidity
            amount = amountStaked[tokenId];

```

## [L-05] Add re-entrancy lock 

 ### Instances
* [FighterFarm.sol #370](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L370)
```solidity
function reRoll(uint8 tokenId, uint8 fighterType) public
```

* [RankedBattle.sol #244](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L244)
```solidity
function stakeNRN(uint256 amount, uint256 tokenId) external

```

## [N-01] An Empty string is passed to the function, avoid hardcoded empty strings
 ### Instances
* [FighterFarm.sol #364](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L364)
```solidity
        _safeTransfer(from, to, tokenId, "");

```





## [N-02] Save String as a constant instead of hardcoding it in the function
 ### Instances
* [FighterFarm.sol #396](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L395C1-L397C6)
```solidity
    function contractURI() public pure returns (string memory) {
        return "ipfs://bafybeifztjs4yuwhqi7bvzhw2ufksynkoiwxss2gnti6j4v25l7iwz7y44";
    }
```
*  [GameItems.sol #95](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L95)
```solidity
    constructor(address ownerAddress, address treasuryAddress_) ERC1155("https://ipfs.io/ipfs/") 

```
* [GameItems.sol #250](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L250)
```solidity
 function contractURI() public pure returns (string memory) {
        return "ipfs://bafybeih3witscmml3padf4qxbea5jh4rl2xp67aydqvqsxmyuzipwtpnii";
    }
```
## [N-03] Function should be allowed to be called only once
 ### Instances
* [GameItems.sol #139](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L139)
```solidity
    function instantiateNeuronContract(address nrnAddress) external {
        require(msg.sender == _ownerAddress);
        _neuronInstance = Neuron(nrnAddress);
    }
```



## [N-04] Avoid on-chain computation

 ### Instances
* [RankedBattle.sol #157](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L157)
```solidity
        rankedNrnDistribution[0] = 5000 * 10**18;

```

## [N-05] Function is so clumped 
The function mentioned below is very messy and needs some cleaning
### Instance
* [RankedBattle.sol #416-500](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L416C1-L500C6)
```solidity
    function _addResultPoints(
        uint8 battleResult, 
        uint256 tokenId, 
        uint256 eloFactor, 
        uint256 mergingPortion,
        address fighterOwner
    ) 
```
### Mitigation
* Consider splitting the logic of the function to make it clearer
