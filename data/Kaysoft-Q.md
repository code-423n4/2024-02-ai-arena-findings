## [L-1] Consider using 2 step ownership transfer

The Neuron.sol#transferOwnership() function transfers ownership in a single step. This can cause the ownership of the contract to be easily lost when a wrong address is mistakenly entered. 

With 2 step ownership transfer, the above mistake can be recovered from because the new owner send a confirmation trasnaction to accept ownership while its pending.

there 7 instances of these
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85C5-L88C6
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L61C5-L64C6
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L120
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L108
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L89
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L167
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L64

```
File: Neuron.sol
function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
```

Recommendation:
Consider using Openzeppelin's Ownable2step library.


## [L-2] Consider having functions that can also remove `minter`, `staker` and `spender`

The `addMinter`, `addStaker` and `addSpender` function are used to grant roles to some contracts. `owner` should be able to also `revoke` roles.

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L93C5-L112C6

Recommendation:
Consider creating functions that allows revoking the above roles. 


## [L-3] FighterFarm.sol has function to add an address as `staker` but no `removeStaker` function.

The `addStaker` function on the FighterFarm.sol is used to add a new address that is allowed to stake fighters on behalf of users.

However, in a situation where a `staker` address is compromised or there is suspicion of compromise, there is no way to remove the address as a `staker`

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L139C5-L142C6
```
/// @notice Adds a new address that is allowed to stake fighters on behalf of users.
    /// @dev Only the owner address is authorized to call this function.
    /// @param newStaker The address of the new staker
    function addStaker(address newStaker) external {
        require(msg.sender == _ownerAddress);
        hasStakerRole[newStaker] = true;
    }
```

## [L-4] Emit event for useful state update.

Most functions and constructors that update state do not emit event 
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L104C5-L222C6
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L95C5-L142C6
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L71C3-L132C6
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L68C5-L134C6
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L146C5-L242C6
- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L60C5-L83C10

Frontends and offchain monitoring tools can be used to listen to events in order to take action when unusual events are emitted when they are not expected.

Recommendation:
Emit event for useful state updates
