## Gas Optimization Issues

| ID     | Finding                                                                              |
| ------ | ------------------------------------------------------------------------------------ |
| [G-01] | Redundant `attributeProbabilities` Initialization in the `AiArenaHelper` Constructor |
| [G-02] | Utilize Cached Variable in Require Statement Checks                                  |
| [G-03] | Use `_spendAllowance` in `Neuron.burnFrom`                                           |
| [G-04] | Omit Redundant Allowance Check in `Neuron.claim`                                     |
| [G-05] | Cache Array Length in Loops                                                          |
| [G-06] | Reuse Cached Variables                                                               |
| [G-07] | Avoid Modifying Storage Variables with Zero                                          |

### [G-01] Redundant `attributeProbabilities` Initialization in the `AiArenaHelper` Constructor

GitHub: [Link](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L49), [Link](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L137)

The deployment cost is unnecessarily increased due to the `probabilities` being set twice in the `attributeProbabilities` mapping within the constructor: initially through the [addAttributeProbabilities](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L137) function and then within a [for loop](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L49). Therefore, it's advised to eliminate the `addAttributeProbabilities` call from the constructor.

Additionally, incorporate the `require(probabilities.length == 6, "Invalid number of attribute arrays");` statement in the constructor for validation.

### [G-02] Utilize Cached Variable in Require Statement Checks

In the `AiArenaHelper.addAttributeDivisor`, utilize the cached [attributesLength](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L72) for the `require` statement check to decrease gas costs.

In the `MergingPool.pickWinner`, utilize the cached [winnersLength](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L122) for the `require` statement check to decrease gas costs.

### [G-03] Use `_spendAllowance` in `Neuron.burnFrom`

Utilizing `ERC20._spendAllowance` can reduce gas costs due to its efficiency.

#### Recommendation

```diff
function burnFrom(address account, uint256 amount) public virtual {
-   require(
-       allowance(account, msg.sender) >= amount,
-       "ERC20: burn amount exceeds allowance"
-   );
-   uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
+   _spendAllowance(account, msg.sender, amount);
    _burn(account, amount);
-   _approve(account, msg.sender, decreasedAllowance);
}
```

### [G-04] Omit Redundant Allowance Check in `Neuron.claim`

The [allowance check](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L139-L142) is already performed within `ERC20.transferFrom` -> `ERC20._spendAllowance`, making it unnecessary.

[ERC20.\_spendAllowance](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/b438cb695a1ac520cee6678610b161b1d5df4d9c/contracts/token/ERC20/ERC20.sol#L337)

```solidity
function _spendAllowance(
    address owner,
    address spender,
    uint256 amount
) internal virtual {
    uint256 currentAllowance = allowance(owner, spender);
    if (currentAllowance != type(uint256).max) {
        require(currentAllowance >= amount, "ERC20: insufficient allowance");
        unchecked {
            _approve(owner, spender, currentAllowance - amount);
        }
    }
}
```

### [G-05] Cache Array Length in Loops

In [FighterFarm.redeemMintPass](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L233), caching `mintpassIdsToBurn.length` will decrease gas costs.

#### Recommendation

```diff
function redeemMintPass(
    uint256[] calldata mintpassIdsToBurn,
    uint8[] calldata fighterTypes,
    uint8[] calldata iconsTypes,
    string[] calldata mintPassDnas,
    string[] calldata modelHashes,
    string[] calldata modelTypes
)  external {
+   uint256 mintpassIdsToBurnLength = mintpassIdsToBurn.length;

    require(
-       mintpassIdsToBurn.length == mintPassDnas.length &&
+       mintpassIdsToBurnLength == mintPassDnas.length &&
        mintPassDnas.length == fighterTypes.length &&
        fighterTypes.length == modelHashes.length &&
        modelHashes.length == modelTypes.length
    );
-   for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
+   for (uint16 i = 0; i < mintpassIdsToBurnLength; i++) {
        require(msg.sender

 == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
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
}
```

### [G-06] Reuse Cached Variables

In [RankedBattle.updateBattleRecord](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L322), a variable [stakeAtRisk](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L341) for the specified `tokenId` is already cached. However, `_addResultPoints` also makes an external call to `stakeAtRiskInstance`. To decrease gas costs, pass the `stakeAtRisk` as a parameter to `_addResultPoints`.

```diff
function updateBattleRecord(
    uint256 tokenId,
    uint256 mergingPortion,
    uint8 battleResult,
    uint256 eloFactor,
    bool initiatorBool
) external {
    ...
    uint256 stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
    if (amountStaked[tokenId] + stakeAtRisk > 0) {
-       _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner);
+       _addResultPoints(battleResult, tokenId, eloFactor, mergingPortion, fighterOwner, stakeAtRisk);
    }
    ...
}
```

### [G-07] Avoid Modifying Storage Variables with Zero

Not modifying storage variables with a zero value saves gas.

Recommendation:

Update points after `points > 0` check.

```diff
function _addResultPoints(
    uint8 battleResult,
    uint256 tokenId,
    uint256 eloFactor,
    uint256 mergingPortion,
    address fighterOwner
) private {
    ...
    if (battleResult == 0) {
       ...

        /// Add points to the fighter for this round
-       accumulatedPointsPerFighter[tokenId][roundId] += points;
-       accumulatedPointsPerAddress[fighterOwner][roundId] += points;
-       totalAccumulatedPoints[roundId] += points;
        if (points > 0) {
+           accumulatedPointsPerFighter[tokenId][roundId] += points;
+           accumulatedPointsPerAddress[fighterOwner][roundId] += points;
+           totalAccumulatedPoints[roundId] += points;
            emit PointsChanged(tokenId, points, true);
        }
    } else if (battleResult == 2) {
        ...
        if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
            ...
-           accumulatedPointsPerFighter[tokenId][roundId] -= points;
-           accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
-           totalAccumulatedPoints[roundId] -= points;
            if (points > 0) {
+               accumulatedPointsPerFighter[tokenId][roundId] -= points;
+               accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
+               totalAccumulatedPoints[roundId] -= points;
                emit PointsChanged(tokenId, points, false);
            }
        }
        ...
    }
}
```
