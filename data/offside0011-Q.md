# Low Risk and Non-Critical Issues

# [01] missing document of `generation` @param

```solidity
File: src\AiArenaHelper.sol
165:      /// @dev Convert DNA and rarity rank into an attribute probability index.
166:      /// @param attribute The attribute name.
167:      /// @param rarityRank The rarity rank.
168:      /// @return attributeProbabilityIndex attribute probability index.
169:     function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
```


# [02] Inadequate randomness

`dna` generation is controlled by `msg.sender`. User can fuzz the sender address to gain extra privilege by generating more competetive `dna`s.

```solidity
File: src\FighterFarm.sol
379:             uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
380:             (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
381:             fighters[tokenId].element = element;
382:             fighters[tokenId].weight = weight;
383:             fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
384:                 newDna,
385:                 generation[fighterType],
386:                 fighters[tokenId].iconsType,
387:                 fighters[tokenId].dendroidBool
388:             );

```

# [03] Unchecked return value of `transferFrom`

It is crucial to adhere to best practices by verifying the return value when transferFrom() encounters failures in specific scenarios.

```solidity
File: src\Neuron.sol
136:     /// @notice Claims the specified amount of tokens from the treasury address to the caller's address.
137:     /// @param amount The amount to be claimed
138:     function claim(uint256 amount) external {
139:         require(
140:             allowance(treasuryAddress, msg.sender) >= amount, 
141:             "ERC20: claim amount exceeds allowance"
142:         );
143:         transferFrom(treasuryAddress, msg.sender, amount);
144:         emit TokensClaimed(msg.sender, amount);
145:     }

```


# [04] Bypass the emission of the event `TokensClaimed` 

The current implementation does not enforce users to utilize the `claim()` function to receive their airdrops. Instead, they can directly invoke the `transferFrom()` function, thereby bypassing the emission of the TokensClaimed event. It is important to notify upstream applications that rely on this event about this behavior.

```solidity
File: src\Neuron.sol
136:     /// @notice Claims the specified amount of tokens from the treasury address to the caller's address.
137:     /// @param amount The amount to be claimed
138:     function claim(uint256 amount) external {
139:         require(
140:             allowance(treasuryAddress, msg.sender) >= amount, 
141:             "ERC20: claim amount exceeds allowance"
142:         );
143:         transferFrom(treasuryAddress, msg.sender, amount);
144:         emit TokensClaimed(msg.sender, amount);
145:     }

```

# [05] No access control on emitting `FighterCreated` event

Sincde the function is public, anybody can trigger this event. It is important to notify upstream applications that rely on this event about this behavior.

```solidity
File: src\FighterOps.sol
52:     /// @notice Emits a FighterCreated event.
53:     function fighterCreatedEmitter(
54:         uint256 id,
55:         uint256 weight,
56:         uint256 element,
57:         uint8 generation
58:     ) 
59:         public 
60:     {
61:         emit FighterCreated(id, weight, element, generation);
62:     }
63: 
```
