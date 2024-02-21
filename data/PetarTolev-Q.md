## Low Risk Findings Overview

| ID     | Finding                                                             |
| ------ | ------------------------------------------------------------------- |
| [L-01] | Staker Role Cannot Be Revoked                                       |
| [L-02] | `getFighterPoints` only returns the points for figter with id 1     |
| [L-03] | The `Neuron` Contract Uses a Deprecated `AccessControl` Function    |
| [L-04] | Unstaked transferred fighter is unusable in the same roundId        |
| [L-05] | Loss of NRNs on token transfer failure in `RankedBattle.unstakeNRN` |
| [L-06] | Possible Cross-Chain Replay Attack in `FighterFarm.claimFighters`   |
| [L-07] | The user can choose fighters' attributes in their favor             |

## Non-critical Findings Overview

| ID      | Finding                                                         |
| ------- | --------------------------------------------------------------- |
| [NC-01] | Implement `_ableToTransfer` in the `ERC20._beforeTokenTransfer` |
| [NC-02] | Use OpenZeppelin's `AccessControl` for Role Management          |
| [NC-03] | Use OpenZeppelin's `Ownable` for Ownership Management           |
| [NC-04] | In `Neuron` Contract Utilize `AccessControl` Functionality      |
| [NC-05] | In `Neuron` Contract, Inherit OpenZeppelin's `ERC20Burnable`    |
| [NC-06] | Duplicated Validations Should Be Refactored to a Function       |

---

# Low Risk Findings

### [L-01] Staker Role Cannot Be Revoked

In `FighterFarm`, the staker role has permission to lock/unlock fighters, determining their transferability.

```solidity
function updateFighterStaking(uint256 tokenId, bool stakingStatus) external {
    require(hasStakerRole[msg.sender]);
    fighterStaked[tokenId] = stakingStatus;
    ...
}
```

```solidity
function _ableToTransfer(uint256 tokenId, address to) private view returns(bool) {
    return (
      _isApprovedOrOwner(msg.sender, tokenId) &&
      balanceOf(to) < MAX_FIGHTERS_ALLOWED &&
      !fighterStaked[tokenId]
    );
}
```

However, if a staker behaves maliciously, the role cannot be revoked.

#### Recommendation

```diff
function addStaker(address newStaker, bool access) external {
    require(msg.sender == _ownerAddress);
-   hasStakerRole[newStaker] = true;
+   hasStakerRole[newStaker] = access;
}
```

### [L-02] `getFighterPoints` only returns the points for figter with id 1

[MergingPool.getFighterPoints](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L206) function should return the points for the fighters from 0 to `maxId`. However, the [points](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L210) array returned from the function is initialized with length of 1, thus the function always reverts for `maxId` > 1.

According to the NatSpec, it should return an aggregated points array.

> /// @notice Retrieves the points for multiple fighters up to the specified maximum token ID.

#### Recomendation

```diff
function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) {
-   uint256[] memory points = new uint256[](1);
+   uint256[] memory points = new uint256[](maxId);
    ...
}
```

### [L-03] The `Neuron` Contract Uses a Deprecated `AccessControl` Function

In `Neuron`, the functions [addMinter](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L95), [addStaker](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L103), [addSpender](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L111) use the deprecated `_setupRole` function.

[OpenZeppelin DOCS](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/b438cb695a1ac520cee6678610b161b1d5df4d9c/contracts/access/AccessControl.sol#L203)

#### Recommendation

Instead of `_setupRole`, use `_grantRole`.

### [L-04] Unstaked transferred fighter is unusable in the same roundId

When a user unstakes their NRNs and then decides to sell/transfer their fighter to another user, they must wait until the next round to restake and enter the ranked battle.

POC:

1. Alice has fighter 1 and stakes 100 NRNs.
2. Alice unstakes 100 NRNs.
3. Alice sells fighter 1 to Bob on OpenSea.
4. Bob cannot stake NRNs, thus cannot enter the ranked battle in the same round.

[unstakeNRN](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L282) sets `hasUnstaked` to true.

[stakeNRN](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L248) check only if `tokenId` and `roundId` have been unstaked.

```solidity
    require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");
```

#### Recommendation

Consider revising the logic to allow newly transferred fighters to participate in the same round.

### [L-05] Loss of NRNs on token transfer failure in `RankedBattle.unstakeNRN`

If the NRN transfer fails, the user will lose their NRNs.

#### Recommendation

```diff
function unstakeNRN(uint256 amount, uint256 tokenId) external {
   ...
    bool success = _neuronInstance.transfer(msg.sender, amount);
+   require(success, "Transfer failed.");
-   if (success) {
        if (amountStaked[tokenId] == 0) {
            _fighterFarmInstance.updateFighterStaking(tokenId, false);
        }
        emit Unstaked(msg.sender, amount);
-   }
}
```

### [L-06] Possible Cross-Chain Replay Attack in `FighterFarm.claimFighters`

The [FighterFarm.claimFighters](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L191) function does not use any nonce or chainId in the `msgHash` calculation.

#### Recommendation

Utilize EIP-155 to include chainId in the `msgHash` calculation.

### [L-07] The user can choose fighters' attributes in their favor

The user can pre-calculate their fighter's attributes in advance and even select the desired ones.

The fighter's base is [calculated](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L470-L472) based on the `dna`, which is provided as input by the user in [redeemMintPass](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L254) and then passed to the [`_createFighterBase`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L499-L501) method. This allows every user to control which fighter they will receive.

# Non-critical Findings

### [NC-01] Implement `_ableToTransfer` in the `ERC20._beforeTokenTransfer`

Move the `_ableToTransfer` check to the `_beforeTokenTransfer` hook in `FighterFarm`.

### [NC-02] Use OpenZeppelin's `AccessControl` for Role Management

Replace the current ownership logic in the contracts with OpenZeppelin's AccessControl. This will help reduce the contracts' complexity and size. Remove `transferOwnership`, `adjustAdminAccess`, `_ownerAddress`, and `isAdmin`.

Instances: [FighterFarm](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol), [GameItems](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol), [MergingPool](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol), [RankedBattle](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol), [VoltageManager](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol).

### [NC-03] Use OpenZeppelin's `Ownable` for Ownership Management

Replace the current ownership logic in the `AiArenaHelper` with OpenZeppelin's `Ownable`. This will help reduce the contract's complexity and size.

### [NC-04] In `Neuron` Contract Utilize `AccessControl` Functionality

1. Replace the `hasRole` checks with `onlyRole` modifiers.
2. Set the `_ownerAddress` as `DEFAULT_ADMIN_ROLE`.
3. Remove the `isAdmin` mapping, instead grant admins the `ADMIN_ROLE`.

This will decrease the contract's size and complexity.

### [NC-05] In `Neuron` Contract, Inherit OpenZeppelin's `ERC20Burnable`

To decrease the `Neuron` contract's size and complexity, inherit from `ERC20Burnable` and remove the [burn](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L161-L165) and [burnFrom](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L192-L204) functions.

### [NC-06] Duplicated Validations Should Be Refactored to a Function

Consider the refactoring of duplicated replenish time validations in `GameItems`.

Recommendations:

```diff
    function mint(uint256 tokenId, uint256 quantity) external {
        require(tokenId < _itemCount);
        uint256 price = allGameItemAttributes[tokenId].itemPrice * quantity;
        require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");
        require(
            allGameItemAttributes[tokenId].finiteSupply == false ||
            (
                allGameItemAttributes[tokenId].finiteSupply == true &&
                quantity <= allGameItemAttributes[tokenId].itemsRemaining
            )
        );
        require(
-           dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp ||
+           isReplenishTimePassed(msg.sender, tokenId) ||
            quantity <= allowanceRemaining[msg.sender][tokenId]
        );

        _neuronInstance.approveSpender(msg.sender, price);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, price);
        if (success) {
-           if (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp) {
+           if (isReplenishTimePassed(msg.sender, tokenId)) {
                _replenishDailyAllowance(tokenId);
            }
            allowanceRemaining[msg.sender][tokenId] -= quantity;
            if (allGameItemAttributes[tokenId].finiteSupply) {
                allGameItemAttributes[tokenId].itemsRemaining -= quantity;
            }
            _mint(msg.sender, tokenId, quantity, bytes("random"));
            emit BoughtItem(msg.sender, tokenId, quantity);
        }
    }

    function getAllowanceRemaining(address owner, uint256 tokenId) public view returns (uint256) {
        uint256 remaining = allowanceRemaining

[owner][tokenId];
-       if (dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp) {
+       if (isReplenishTimePassed(owner, tokenId)) {
            remaining = allGameItemAttributes[tokenId].dailyAllowance;
        }
        return remaining;
    }

+   function isReplenishTimePassed(address owner, uint256 tokenId) internal view returns(bool) {
+       return dailyAllowanceReplenishTime[owner][tokenId] <= block.timestamp;
+   }
```
