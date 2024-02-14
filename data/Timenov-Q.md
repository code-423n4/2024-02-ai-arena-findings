# AI Arena QA report by Timenov

## Summary

| |Issue|Instances|
 |-|:-|:-:|
| I-01 | State variable can be set in constructor. | 6 |
| I-02 | Process failed transfers. | 7 |
| I-03 | No way to change _neuronInstance. | 1 |
| L-01 | Wrong ERC1155 uri. | 1 |
| L-02 | Function will return wrong value if item has infinite supply. | 1 |
| L-03 | Mint will never get to the MAX_SUPPLY. | 1 |

### [I-01] State variables can be set in constructor.
In some contracts, state variable that are referencing other contracts are set in functions(usually their names start with `instantiate`). This can be removed so that the deployer will not need to call these functions on deployment, but only when these variables need to be updated. How each contract can be optimized:

- FighterFarm.sol -> Set `_aiArenaHelperInstance` and `_neuronInstance` in the constructor.
- GameItems.sol -> Set `_neuronInstance` in the constructor.
- MergingPool.sol -> Set `_fighterFarmInstance` in the constructor(this must be deployed after `FighterFarm` contract).
- RankedBattle.sol -> Set `_neuronInstance` and `_mergingPoolInstance` in the constructor.

The other contracts have done this already. Also consider renaming all the functions that will be affected from these changes. Currently they start with `instantiate`. A better naming will be `set` or `update`. All these changes will increase the code readability, remove deployment errors and cost less gas on deployment.

### [I-02] Process failed transfers.
The boolean `success` is taken after `NRN` transfer. However only the `true` value is processed. Consider reverting when the result is `false` so that the users are notified when the token transfer has failed.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L376

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L164

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L251

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L283

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L493

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L80

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L100

### [I-03] No way to change _neuronInstance.
In all contracts that have state variable `_neuronInstance` that references to the `Neuron` contract, have functions to change the address. However in `StakeAtRisk.sol` this is not true. The variable there is set in the constructor and there is no other function to allow the admin to change it.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L67

### [L-01] Wrong ERC1155 uri.
In `GameItems.sol` constructor the uri value `https://ipfs.io/ipfs/` is passed to the `ERC1155`. This does not seem to be valid uri as we can see the comment in `ERC1155.sol`.

```solidity
    // Used as the URI for all token types by relying on ID substitution, e.g. https://token-cdn-domain/{id}.json
    string private _uri;
```

There is no way to change this after deployment.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L95

### [L-02] Function will return wrong value if item has infinite supply.
The function `remainingSupply` in `GameItems` is used to return the remaining supply of a game item.

```solidity
    function remainingSupply(uint256 tokenId) public view returns (uint256) {
        return allGameItemAttributes[tokenId].itemsRemaining;
    }
```

This can be misleading if `!allGameItemAttributes[tokenId].finiteSupply`. In this case the `itemsRemaining` does not matter. Consider rewriting the function to avoid this type of confusion.

```solidity
    function remainingSupply(uint256 tokenId) public view returns (uint256) {
        return allGameItemAttributes[tokenId].finiteSupply 
            ? allGameItemAttributes[tokenId].itemsRemaining 
            : type(uint256).max;
    }
```

### [L-03] Mint will never get to the MAX_SUPPLY.
The `mint()` in `Neuron.sol` will not allow `totalSupply == MAX_SUPPLY`. This means 1 token will never be minted.

```solidity
    function mint(address to, uint256 amount) public virtual {
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L155-L159

This is because a strict check is used `<` instead of `<=`. Add the following test case to `Neuron.t.sol`:

```solidity
    function testDoesNotMintWholeSupply() public {
        address minter = vm.addr(3);
        _neuronContract.addMinter(minter);
        uint256 difference = _neuronContract.MAX_SUPPLY() - _neuronContract.totalSupply();

        vm.prank(minter);
        vm.expectRevert("Trying to mint more than the max supply");
        _neuronContract.mint(minter, difference);
    }
```

Run `forge test --match-contract Neuron --match-test testDoesNotMintWholeSupply`. Test passes:

```solidity
[PASS] testDoesNotMintWholeSupply() (gas: 41181)
```