
# [G-01] Pass empty strings for irrelevant parameters

`bytes("random")` can be replaced with `bytes("")`

```solidity
File: src\GameItems.sol
173:             _mint(msg.sender, tokenId, quantity, bytes("random"));

```

# [G-02] Downcasting cost extra gas

`dailyAllowanceReplenishTime` maps to uint256, the downcasting here cost more.

```solidity 
File: src\GameItems.sol
312:     function _replenishDailyAllowance(uint256 tokenId) private {
313:         allowanceRemaining[msg.sender][tokenId] = allGameItemAttributes[tokenId].dailyAllowance;
314:         dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
315:     }    

```

# [G-03] Inactive (unnecessary) function overriding

The following function overrides do nothing but cost extra gas.

```solidity
File: src\FighterFarm.sol
406:     /// @notice Returns whether a given interface is supported by this contract.
407:     /// @dev Calls ERC721.supportsInterface.
408:     /// @param _interfaceId The interface ID.
409:     /// @return Bool whether the interface is supported by this contract.
410:     function supportsInterface(bytes4 _interfaceId)
411:         public
412:         view
413:         override(ERC721, ERC721Enumerable)
414:         returns (bool)
415:     {
416:         return super.supportsInterface(_interfaceId);
417:     }

```

```solidity
File: src\FighterFarm.sol
443:     /// @notice Hook that is called before a token transfer.
444:     /// @param from The address transferring the token.
445:     /// @param to The address receiving the token.
446:     /// @param tokenId The ID of the NFT being transferred.
447:     function _beforeTokenTransfer(address from, address to, uint256 tokenId)
448:         internal
449:         override(ERC721, ERC721Enumerable)
450:     {
451:         super._beforeTokenTransfer(from, to, tokenId);
452:     }

```

# [G-04] Array length check is redundant

Solidity protects on array index overflow.

```solidity
File: src\FighterFarm.sol
243:         require(
244:             mintpassIdsToBurn.length == mintPassDnas.length && 
245:             mintPassDnas.length == fighterTypes.length && 
246:             fighterTypes.length == modelHashes.length &&
247:             modelHashes.length == modelTypes.length
248:         );

```

In this project, there appears to be an excessive use of array length checks. It's important to note that Solidity inherently prevents arrays from overflowing by default, and invalid inputs will automatically cause the function to revert. Therefore, these additional array length checks are unnecessary and can be safely removed.
