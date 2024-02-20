# QA Report

## Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01) | Previous owner retains administrative abilities after ownership transfer | Low |
| [L-02](#l-02) | The `Neuron::burnFrom()` does not support infinite approval | Low |
| [L-03](#l-03) | Lack of setter functions for crucial external Addresses | Low |
| [L-04](#l-04) | Lack of role revocation functionality in `Neuron` Contract | Low |
| [L-05](#l-05) | Lack of resetting accessible of crucial roles | Low |
| [L-06](#l-06) | Lack of customized replenishment times for game items | Low |
| [N-01](#n-01) | Recommend explicitly setting the `GameItems::itemsRemaining` when the supply is not finite | Non-Critical |
| [N-02](#n-02) | Enhance consistency in the `GameItems::remainingSupply()` when querying items of non-finite supply | Non-Critical |
</br>


## **[L-01] Previous owner retains administrative abilities after ownership transfer**<a name="l-01"></a>

As the first owner retrieves the admin accessibility at the deployment of the contract, however, the `transferOwnership()` does not reset the admin accessibility of the previous owner when transferring ownership. 

This leads to the previous owner still retaining administrative abilities until the new owner resets their accessibility via `adjustAdminAccess()`.

*There is 5 instances of this issue:*

```solidity
File: GameItems.sol

    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
```

```solidity
File: MergingPool.sol

    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
```

```solidity
File: Neuron.sol

    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
```

```solidity
File: RankedBattle.sol

    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
```

```solidity
File: VoltageManager.sol

    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
```

### Recommendation
Update the `transferOwnership()` to reset the admin accessibility of the previous owner by adding a line of code to revoke their admin access.

The functions that should be updated for this issue are:
* [GameItems::transferOwnership()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L108-L111)
* [MergingPool::transferOwnership()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L89-L92)
* [Neuron::transferOwnership()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L85-L88)
* [RankedBattle::transferOwnership()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L167-L170)
* [VoltageManager::transferOwnership()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L64-L67)

The example recommendation code: 

```diff
File: GameItems.sol

        function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
+       isAdmin[_ownerAddress] = false;
        _ownerAddress = newOwnerAddress;
    }
```
</br>

---

## **[L-02] The `Neuron::burnFrom()` does not support infinite approval**<a name="l-02"></a>

The [Neuron::burnFrom()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L196-L204) does not support infinite approval, which is inconsistent with the behavior of [ERC20::transferFrom()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/token/ERC20/ERC20.sol#L158-L167) when the spender performs on behalf of the token owner.

*There is one instance of this issue:*

```solidity
File: Neuron.sol

    function burnFrom(address account, uint256 amount) public virtual {
        require(
            allowance(account, msg.sender) >= amount, 
            "ERC20: burn amount exceeds allowance"
        );
        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
        _burn(account, amount);
        _approve(account, msg.sender, decreasedAllowance);
    }
```

### Recommendation
Implement the functionality to support infinite approval within the `Neuron::burnFrom()` function, and also refactoring the function to consume gas more efficiently.

```diff
File: Neuron.sol

    function burnFrom(address account, uint256 amount) public virtual {
+        uint256 currentAllowance = allowance(account, msg.sender);
+        if (currentAllowance != type(uint256).max) {
            require(
-                allowance(account, msg.sender) >= amount
+                currentAllowance >= amount,
                "ERC20: burn amount exceeds allowance"
            );
+           unchecked {
+               _approve(account, msg.sender, currentAllowance - amount);
+           }
+       }
-       uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
        _burn(account, amount);
-       _approve(account, msg.sender, decreasedAllowance);
    }
```
</br>

---

## **[L-03] Lack of setter functions for crucial external Addresses**<a name="l-03"></a>

The `treasuryAddress_` and `delegatedAddress` addresses are external contracts used to retrieve the NRN and act as signers of the claim fighter messages, respectively.

These two states lack setter functions, which means their addresses cannot be updated. Considering these addresses could potentially be compromised, adding the ability to update them would mitigate this risk.

*There is 4 contracts of this issue:*

* The [FighterFarm](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol) contract
* The [GameItems](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol) contract
* The [Neuron](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol) contract
* The [StakeAtRisk](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol) contract

### Recommendation
Implementing setter functions for the `treasuryAddress_` and `delegatedAddress` states to allow for their update is recommended. 

These setter functions should be restricted to high-level roles to ensure proper control over the modification of these critical addresses.

The contracts that should be updated for this issue are:
* The [FighterFarm](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol) contract
* The [GameItems](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol) contract
* The [Neuron](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol) contract
* The [StakeAtRisk](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/StakeAtRisk.sol) contract

</br>

---

## **[L-04] Lack of role revocation functionality in `Neuron` Contract**<a name="l-04"></a>

The [Neuron](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol) contract does not designate an account holding the[AccessControl::DEFAULT_ADMIN_ROLE](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.0/contracts/access/AccessControl.sol#L57).

Therefore, if the contract owner mistakenly assigns roles such as `MINTER_ROLE`, `STAKER_ROLE`, or `SPENDER_ROLE` to unintended addresses, there is **no mechanism to revoke these roles from those addresses**.


```solidity
--> function revokeRole(bytes32 role, address account) public virtual override onlyRole(getRoleAdmin(role)) {
        _revokeRole(role, account);
    }
```

*There is one instance of this issue:*

```solidity
File: Neuron.sol

    contract Neuron is ERC20, AccessControl {

        // SNIPPED

    }
```

### Recommendation
Assign an account holding the `AccessControl::DEFAULT_ADMIN_ROLE` in the `Neuron` contract to enable role management.

Additionally, implement logic to revoke the `AccessControl::DEFAULT_ADMIN_ROLE` when transferring ownership

```diff
File: Neuron.sol

    constructor(address ownerAddress, address treasuryAddress_, address contributorAddress)
        ERC20("Neuron", "NRN")
    {
        _ownerAddress = ownerAddress;
        treasuryAddress = treasuryAddress_;
        isAdmin[_ownerAddress] = true;
        _mint(treasuryAddress, INITIAL_TREASURY_MINT);
        _mint(contributorAddress, INITIAL_CONTRIBUTOR_MINT);
+       _grantRole(DEFAULT_ADMIN_ROLE, _ownerAddress);
    } 

    //SNIPPED

    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
+       _revokeRole(DEFAULT_ADMIN_ROLE, _ownerAddress);
        _ownerAddress = newOwnerAddress;
+       _grantRole(DEFAULT_ADMIN_ROLE, newOwnerAddress);
    }
```
</br>

---

## **[L-05] Lack of resetting accessible of crucial roles**<a name="l-05"></a>

The [FighterFarm](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol) and [GameItems](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol) contracts do not designate an address holding the staker role `hasStakerRole` and burning role `allowedBurningAddresses`, respectively.

*There is 2 instances of this issue:*

```solidity
File: FighterFarm.sol

    function addStaker(address newStaker) external {
        require(msg.sender == _ownerAddress);
-->     hasStakerRole[newStaker] = true;
    }
```

```solidity
File: GameItems.sol

    function setAllowedBurningAddresses(address newBurningAddress) public {
        require(isAdmin[msg.sender]);
-->     allowedBurningAddresses[newBurningAddress] = true;
    }
```

### Recommendation
Update the logic to flexibly set the accessible status as follows:

```diff
File: FighterFarm.sol

-   function addStaker(address newStaker) external {
+   function setStaker(address newStaker, bool status) external {
        require(msg.sender == _ownerAddress);
-       hasStakerRole[newStaker] = true;
+       hasStakerRole[newStaker] = status;
    }
```

```diff
File: GameItems.sol

-    function setAllowedBurningAddresses(address newBurningAddress) public {
+    function setAllowedBurningAddresses(address newBurningAddress, bool status) public {
        require(isAdmin[msg.sender]);
-       allowedBurningAddresses[newBurningAddress] = true;
+       allowedBurningAddresses[newBurningAddress] = status;
    }
```
</br>

---

## **[L-06] Lack of customized replenishment times for game items**<a name="l-06"></a>

Each game item should have its replenishment time specified, taking into account factors such as the item's rarity or special abilities. This ensures that the time required to replenish each item is customized to its unique characteristics within the game.

*There is one instance of this issue:*

```solidity
File: GameItems.sol

function _replenishDailyAllowance(uint256 tokenId) private {
        allowanceRemaining[msg.sender][tokenId] = allGameItemAttributes[tokenId].dailyAllowance;
-->     dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
    }
```

### Recommendation
Update the logic to dynamically set the `dailyAllowanceReplenishTime` for each game items:

```diff
File: GameItems.sol

function _replenishDailyAllowance(uint256 tokenId) private {
        allowanceRemaining[msg.sender][tokenId] = allGameItemAttributes[tokenId].dailyAllowance;
-       dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + 1 days);
+       dailyAllowanceReplenishTime[msg.sender][tokenId] = uint32(block.timestamp + allGameItemAttributes[tokenId].dailyAllowanceReplenishPeriod);
    }
```

```diff
File: GameItems.sol

    struct GameItemAttributes {
        string name;
        bool finiteSupply;
        bool transferable;
        uint256 itemsRemaining;
        uint256 itemPrice;
        uint256 dailyAllowance;
+       uint256 dailyAllowanceReplenishPeriod;
    }  
```

```diff
File: GameItems.sol

    function createGameItem(
        string memory name_,
        string memory tokenURI,
        bool finiteSupply,
        bool transferable,
        uint256 itemsRemaining,
        uint256 itemPrice,
        uint16 dailyAllowance,
+       uint256 dailyAllowanceReplenishPeriod,
    ) 
        public 
    {
        require(isAdmin[msg.sender]);
        allGameItemAttributes.push(
            GameItemAttributes(
                name_,
                finiteSupply,
                transferable,
                itemsRemaining,
                itemPrice,
                dailyAllowance,
+               dailyAllowanceReplenishPeriod
            )
        );
        if (!transferable) {
          emit Locked(_itemCount);
        }
        setTokenURI(_itemCount, tokenURI);
        _itemCount += 1;
    }
```
</br>

---

## **[N-01] Recommend explicitly setting the `GameItems::itemsRemaining` when the supply is not finite** <a name="n-01"></a>

The [GameItems::createGameItem()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L208-L235) could enhancing the explicit setup of  `itemsRemaining` when creating non-finite supply items.

*There is one instance of this issue:*

```solidity
File: GameItems.sol

    function createGameItem(
        string memory name_,
        string memory tokenURI,
        bool finiteSupply,
        bool transferable,
        uint256 itemsRemaining,
        uint256 itemPrice,
        uint16 dailyAllowance
    ) 
        public 
    {
        require(isAdmin[msg.sender]);
        allGameItemAttributes.push(
            GameItemAttributes(
                name_,
                finiteSupply,
                transferable,
                itemsRemaining,
                itemPrice,
                dailyAllowance
            )
        );
        if (!transferable) {
          emit Locked(_itemCount);
        }
        setTokenURI(_itemCount, tokenURI);
        _itemCount += 1;
    }
```

### Recommendation
Explicitly set the `GameItems::itemsRemaining` when the supply is not finite to infinite `itemsRemaining` supply.

```diff
File: GameItems.sol

    function createGameItem(
        string memory name_,
        string memory tokenURI,
        bool finiteSupply,
        bool transferable,
        uint256 itemsRemaining,
        uint256 itemPrice,
        uint16 dailyAllowance
    ) 
        public 
    {
        require(isAdmin[msg.sender]);
        allGameItemAttributes.push(
            GameItemAttributes(
                name_,
                finiteSupply,
                transferable,
+               finiteSupply ? itemsRemaining: type(uint256).max,
                itemPrice,
                dailyAllowance
            )
        );
        if (!transferable) {
          emit Locked(_itemCount);
        }
        setTokenURI(_itemCount, tokenURI);
        _itemCount += 1;
    }
```
</br>

---

## **[N-02] Enhance consistency in the `GameItems::remainingSupply()` when querying items of non-finite supply**<a name="n-02"></a>

The [GameItems::remainingSupply()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L279-L281) could be improved to provide clearer support for querying the supply of each item by returning the `finiteSupply` status along with the supply value.

This can explicitly assist in off-chain retrieval of `remainingSupply` of each item and facilitate better decision-making regarding the management of items as a non-finite supply items.

*There is one instance of this issue:*

```solidity
File: GameItems.sol

    function remainingSupply(uint256 tokenId) public view returns (uint256) {
        return allGameItemAttributes[tokenId].itemsRemaining;
    }
```

### Recommendation
Update the setting the `GameItems::remainingSupply()` to return `itemsRemaining` amount along with the item's `finiteSupply` status.

```diff
File: GameItems.sol

    function remainingSupply(uint256 tokenId) public view returns (uint256, bool) {
+       return (allGameItemAttributes[tokenId].itemsRemaining, allGameItemAttributes[tokenId].finiteSupply);
    }
```
</br>

---