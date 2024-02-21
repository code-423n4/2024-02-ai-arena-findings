### [L-1] In `GameItems` the `instantiateNeuronContract` function could lead to revert if not initialized

**Description:** In `GameItems` the `instantiateNeuronContract` function instantiates the NRN contract address. If someone calls `mint` before instantiation, it will revert.

**Impact:** The function `mint` could revert if nrnAddress is not instantiated.

**Recommended Mitigation:** Recommended mitigations are:

1. Be sure as soon as the contract is deployed the `instantiateNeuronContract` function is called with the right address.
2. Instantiate the `_neuronInstance` in the constructor and add a `changeNeuronInstance` function in order to change it.

### [L-2] In `GameItems` the `createGameItem` function only emits an event, only if the item is not transferable and not when it is transferable

**Description:** In `GameItems` the `createGameItem` function is responsible for creating a game item, but an event is emitted only when the item is not transferable. In the contract, there is an event called `Unlocked`, responsible for emitting an event for transferable items. Emit the `Unlocked` event if `transferable` param is equal to true.

**Impact:** Not communicating information on the blockchain if the item created is transferable.

**Recommended Mitigation:** Emit the `Unlocked` event if `transferable` param is equal to true

```diff
    if (!transferable) {
        emit Locked(_itemCount);
+   } else {
+       emit Unlocked(_itemCount);
+    }
```

### [L-3] In `GameItems` the `_replenishDailyAllowance` function casts uint32 to uint256, which can lead to integer overflow

**Description:** In `GameItems` the `_replenishDailyAllowance` function updates the allowances. In the update of `dailyAllowanceReplenishTime` there is an unnecessary cast from uint256 to uint32. Parsing from a higher to a lower integer number is not a recommended practice.

**Impact:** The unsafe cast from uint32 to uint256, can potentially lead to integer overflow

**Recommended Mitigation:** Remove the casting

```diff
- dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
+ dailyAllowanceReplenishTime[msg.sender][tokenId] = block.timestamp + 1 days;
```

### [L-4] In `Neuron` the functions `transferOwnership` and `adjustAdminAccess` do not emit an event after changing storage

**Description:** In `Neuron` the functions `transferOwnership` and `adjustAdminAccess` change the state of the blockchain, therefore should emit an event.

**Impact:** Not communicating information on the blockchain if the ownership is transfered, or an address has been granted the role `Admin`.

**Recommended Mitigation:** Add these events and emit them after the function is executed:

```diff
+ event TransferOwnership(address newOwnerAddress);

function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
+       emit TransferOwnership(newOwnerAddress);
    }

+ event AdminAccessAdjusted(address adminAddress, bool access);

function adjustAdminAccess(address adminAddress, bool access) external {
        require(msg.sender == _ownerAddress);
        isAdmin[adminAddress] = access;
+       emit AdminAccessAdjusted(adminAddress, access)
    }
```

### [L-5] In `MergingPool` the functions `updateWinnersPerPeriod`, `transferOwnership`, and `adjustAdminAccess` do not emit an event after changing storage

**Description:** In `MergingPool` the functions `updateWinnersPerPeriod`, `transferOwnership`, and `adjustAdminAccess` change the state of the blockchain, therefore should emit an event.

**Impact:** Not communicating information on the blockchain if the ownership is transfered, updateWinnersPerPeriod is updated, or an address has been granted the role `Admin`.

**Recommended Mitigation:** Add these events and emit them after the function is executed:

```diff
+ event TransferOwnership(address newOwnerAddress);

function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
+       emit TransferOwnership(newOwnerAddress);
    }

+ event AdminAccessAdjusted(address adminAddress, bool access);

function adjustAdminAccess(address adminAddress, bool access) external {
        require(msg.sender == _ownerAddress);
        isAdmin[adminAddress] = access;
+       emit AdminAccessAdjusted(adminAddress, access)
    }

+ event WinnerPerPeriodUpdate(uint256 newWinnersPerPeriod);

function updateWinnersPerPeriod(uint256 newWinnersPerPeriodAmount) external {
        require(isAdmin[msg.sender]);
        winnersPerPeriod = newWinnersPerPeriodAmount;
+       emit WinnerPerPeriodUpdate(newWinnersPerPeriodAmount);
    }
```

### [L-6] In `FighterFarm` the functions `instantiateAIArenaHelperContract`, `instantiateMintpassContract`, `instantiateNeuronContract`, and `setMergingPoolAddress` could lead to revert if not instantiated

**Description:** In `FighterFarm` the functions `instantiateAIArenaHelperContract`, `instantiateMintpassContract`, `instantiateNeuronContract`, and `setMergingPoolAddress` function instantiate important addresses in the contract. If not called right after the contract deployment, some functions could be unusable.

**Impact:** Not instantiating these addresses could lead to some functions being unusable.

**Recommended Mitigation:** Recommended mitigations are:

1. Be sure as soon as the contract is deployed all functions mentioned above should be called with the right addresses.
2. Instantiate the needed addresses in the constructor and add functions in order to change them.

### [I-1] In `GameItems` the function `setTokenURI` can set the tokenURI to a nonexistent game item

**Description:** In `GameItems` the function `setTokenURI` is responsible for setting the tokenURI to a specific tokenID. However, the function is public and onlyAdmin, meaning that an admin could without intention set a tokenURI to a non-existent tokenId.

**Recommended Mitigation:** Check if the `tokenId` is bigger than the `_itemCount`.

```diff
    function setTokenURI(uint256 tokenId, string memory _tokenURI) public {
+       if (tokenId > _itemCount) revert CustomError();
        require(isAdmin[msg.sender]);
        _tokenURIs[tokenId] = _tokenURI;
    }
```

### [I-2] `GameItems::createGameItem` no check for params `itemPrice` and `dailyAllowance` to be greater than 0

**Description:** In the `GameItems` contract, the function `createGameItem` has no check for important parameters - `itemPrice` and `dailyAllowance`.

**Recommended Mitigation:** Add the following check

```diff
+ if (itemPrice <= 0 && dailyAllowance <= 0) revert CustomError();
```

### [I-3] `FighterOps::viewFighterInfo` returns the address that is passed at `owner`.

**Description:** In contract `FighterOps` the function `viewFighterInfo` returns the address that is passed to it in the parameter `address owner`. The address is not used for any operations and is just returned.

### [I-4] `Neuro::contributorAddress` initialized but never used

**Description:** In contract `Neuron` the `contributorAddress` is supposed to distribute NRNs to early contributors. But the address is only initialized and never used.

### [I-5] `VoltageManager::_ownerAddress` use the private keyword for better readability

**Description:** In contract `VoltageManager` the variable `_ownerAddress` has not been specified visibility. For better readability use the private keyword.

**Recommended Mitigation:** Add the `private` keyword

```diff
    /// The address that has owner privileges (initially the contract deployer).
-    address _ownerAddress;
+    address private _ownerAddress;
```
