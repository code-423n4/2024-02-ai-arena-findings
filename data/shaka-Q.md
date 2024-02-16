# Low Risk Issues

| ID |Title|
|:--:|:-------|
|[L-01]| `MergingPool:getFighterPoints` reverts when called with `maxId` greater than 1 |
|[L-02]| The value of `RankedBattle.totalBattles` will be doubled |
|[L-03]| Signature verification in `FighterFarm.claimFighters` will always pass if `_delegatedAddress` is set to address zero |
|[L-04]| Signature in `FighterFarm.claimFighters` can be replayed in other chains |

## [L-01] `MergingPool:getFighterPoints` reverts when called with `maxId` greater than 1

The size of the array returned by `getFighterPoints` is [set to 1](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L206), so this function can only be used to get the points of fighter with id 0. This makes it not possible to use this function for on-chain integrations with the protocol.

Additionally, name and description of the parameter `maxId` is misleading, as it is not the maximum id of the fighters, but the number of fighters to return points for. E.g. maxId = 1 returns up to fighter with id 0.

Consider applying the following changes:

```diff
File: MergingPool.sol

    function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
-       uint256[] memory points = new uint256[](1);
+       uint256[] memory points = new uint256[](maxId + 1);
-       for (uint256 i = 0; i < maxId; i++) {
+       for (uint256 i = 0; i <= maxId; i++) {
            points[i] = fighterPoints[i];
        }
        return points;
    }
```

## [L-02] The value of `RankedBattle.totalBattles` will be doubled

Each time the game server address calls `updateBattleRecord`, the value of `totalBattles` [is incremented by 1](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L348). As this method should be called twice for battle (once for each fighter), the value of `totalBattles` will be doubled.

Consider sending the data for both fighters in a single call to `updateBattleRecord`.

## [L-03] Signature verification in `FighterFarm.claimFighters` will always pass if `_delegatedAddress` is set to address zero

In the case the `FighterFarm` is deployed setting the `_delegatedAddress` to address zero, the [signature verification](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L206) in the `claimFighters` function will always pass. Given that this misconfiguration could not be evident in the early stages of the protocol, but lead to a high-impact issue (free minting of fighters), it is recommended to add a check in the constructor to prevent the value of `_delegatedAddress` from being set to address zero.

## [L-04] Signature in `FighterFarm.claimFighters` can be replayed in other chains

If the protocol decides to deploy the FighterFarm contract in other chains and use the sale delegated address to sign the messages for claiming fighters, the same signature can be replayed in different chains to claim fighters in all of them.

Consider adding a chain id to the message to prevent replay attacks.

# Non-Critical Issues

| ID |Title|Instances|
|:--:|:-------|:--:|
|[N-01]| Inconsistent use of `attributes` array size | 1 |
|[N-02]| Max NRN supply cannot be reached | 1 |
|[N-03]| Unnecessary code | 2 |
|[N-04]| Incorrect comments | 8 |

## [N-01] Inconsistent use of `attributes` array size 

In the AiArenaHelper contract the size of the [`attributes` array](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L17) is used in different places to check for valid input data and to iterate over the size of the array. In some places it is used as [`attributes.length`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L70) and in other places the value is hardcoded as [`6`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L133).

Consider using a constant to store the size of the array for consistency and gas optimization.

## [N-02] Max NRN supply cannot be reached

The mint function in the Neuron contract has a check to prevent minting more than the max supply, but [the condition](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L156) is incorrect. The condition should be `totalSupply() + amount <= MAX_SUPPLY` instead of `totalSupply() + amount < MAX_SUPPLY`.

## [N-03] Unnecessary code

```solidity
File: AiArenaHelper.sol

41    constructor(uint8[][] memory probabilities) {
42        _ownerAddress = msg.sender;
43
44        // Initialize the probabilities for each attribute
45        addAttributeProbabilities(0, probabilities); 👈 
46
47        uint256 attributesLength = attributes.length;
48        for (uint8 i = 0; i < attributesLength; i++) {
49            attributeProbabilities[0][attributes[i]] = probabilities[i];
50            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i]; 👈 Set twice
51        }
52    } 
    (...)
68    function addAttributeDivisor(uint8[] memory attributeDivisors) external {
69        require(msg.sender == _ownerAddress);
70        require(attributeDivisors.length == attributes.length);
71
72        uint256 attributesLength = attributes.length;
73        for (uint8 i = 0; i < attributesLength; i++) {
74            attributeToDnaDivisor[attributes[i]] = attributeDivisors[i]; 👈 
75        }
76    }    
```

```solidity
File: FighterFarm.sol

443    /// @notice Hook that is called before a token transfer.
444    /// @param from The address transferring the token.
445    /// @param to The address receiving the token.
446    /// @param tokenId The ID of the NFT being transferred.
447    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
448        internal
449        override(ERC721, ERC721Enumerable)
450    {
451        super._beforeTokenTransfer(from, to, tokenId); 👈 Just calls the default implementation
452    }
```

## [N-04] Incorrect comments and naming

```solidity
File: MergingPool.sol

136    /// @param modelURIs The array of model URIs corresponding to each round and winner address. 👈
137    /// @param modelTypes The array of model types corresponding to each round and winner address.
138    /// @param customAttributes Array with [element, weight] of the newly created fighter.
139    function claimRewards(
140        string[] calldata modelURIs, 👈
    (...)
154                   _fighterFarmInstance.mintFromMergingPool(
155                        msg.sender,
156                        modelURIs[claimIndex], 👈 This value is described as `modelHash The hash of the ML model 
                                                     associated with the fighter` in the FighterFarm contract
```

```solidity
File: MergingPool.sol

50    /// @notice Mapping of address to fighter points. 👈 It should say "Mapping of **fighter ID** to points."
51    mapping(uint256 => uint256) public fighterPoints;
```

```solidity
File: FighterFarm.sol

50    /// The address that has owner privileges (initially the contract deployer). 👈 Not contract deployer, but value set in constructor
51    address _ownerAddress;
```

```solidity
File: GameItems.sol

60    /// The address that has owner privileges (initially the contract deployer). 👈 Not contract deployer, but value set in constructor
61    address _ownerAddress;
```

```solidity
File: MergingPool.sol

34    /// The address that has owner privileges (initially the contract deployer). 👈 Not contract deployer, but value set in constructor
35    address _ownerAddress;
```

```solidity
File: Neuron.sol

48    /// The address that has owner privileges (initially the contract deployer). 👈 Not contract deployer, but value set in constructor
49    address _ownerAddress;
```

```solidity
File: RankedBattle.sol

71    /// The address that has owner privileges (initially the contract deployer). 👈 Not contract deployer, but value set in constructor
72    address _ownerAddress;
```

```solidity
File: VoltageManager.sol

22    /// The address that has owner privileges (initially the contract deployer). 👈 Not contract deployer, but value set in constructor
23    address _ownerAddress;
```