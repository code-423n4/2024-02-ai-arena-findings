#[G-01] The `transferOwnership` function does not make the owner an admin

The `transferOwnership` function updates the owner of the contract, but does not give the owner admin functionality, even though the owner is only address that can call `adjustAdminAccess`. This means that there is no reason for an owner to not be an admin, thus the address would have to make a second function call to `adjustAdminAccess` at a gas cost. If there was an incentive for an owner to not be an admin, the current implementation does not stop an owner from making themselves an admin. 

```diff
function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
++ isAdmin[newOwnerAddress] = true;
        _ownerAddress = newOwnerAddress;
    }
```

#[G-02] The `attributeProbabilities` mapping gets overridden in the constructor

The constructor calls `addAttributeProbabilities` which “Initializes the probabilities for each attribute” via a for loop in `addAttributeProbabilities`,  but then overwrites those probabilities and adds them as part of the for loop in the constructor itself. It is, thus not necessary to call `addAttributeProbabilities` in the constructor, thus saving gas. 

Location: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L45

Location: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85-L88

#[G-03] The `defaultAttributeDivisor` can be passed as a parameter in the constructor

The `defaultAttributeDivisor` is only used once in the constructor to populate the initial `attributeToDnaDivisor` .It is not necessary to store an array in storage for a single use during the construction of the contract. Consider passing the array as a constructor parameter. 

Location: https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L50