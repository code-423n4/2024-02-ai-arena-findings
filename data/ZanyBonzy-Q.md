# 1. `useVoltageBattery` function should check for user replenish time.

Lines of code* 

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L93

## Impact

Users have the choice to replenish their voltage either by waiting for the replenish time to end or but using the voltage battery. The `useVoltageBattery` function doesn't check if the replenish time has passed before replenishing the user's voltage. Hence, users unaware can call the function, burning one of their batteries, only to have the voltage replenished again when the `spendVoltage` function is called.

```
    function useVoltageBattery() public { 
        require(ownerVoltage[msg.sender] < 100); 
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0); 
        _gameItemsContractInstance.burn(msg.sender, 0, 1); 
        ownerVoltage[msg.sender] = 100; 
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    } 
```
## Recommended Mitigation Steps
Consider including a check for replenish time in an if else format, so as to save users batteries.
Also, consider reducing the required voltage percentage to something lesser, e.g 10 so as to help users save cost on batteries.
```
    function useVoltageBattery() public { 
        require(ownerVoltage[msg.sender] < 100); 
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) { 
            _replenishVoltage(spender);
        } 
        else{
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0); 
        _gameItemsContractInstance.burn(msg.sender, 0, 1); 
        ownerVoltage[msg.sender] = 100; 
        }
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    } 
```


***

# 2. Transferring ownership doesn't disable admin status for old owner or enable admin status for new owner

Lines of code* 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L146-L158
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L167
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L176
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L95-L99
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L108
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L117
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L71-L79
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L89
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L98
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L68-L76
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L85
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L118
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L51-L55
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L64
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L73

## Impact
In the constructor of the `Neuron`, `GameItems`, `VoltageManager`, `MergingPool`, `RankedBattle`, contracts, the owner address is declared and immediately granted admin acess. However, upon transfer of ownership to another address, the new owner isn't immediately granted admin access, neither is the old owner's admin access revoked. The owner has to manually revoke the admin access for the previous owner and grant himself owner.
For example, in the `VoltageManager` contract

```
    constructor(address ownerAddress, address gameItemsContractAddress) {
        _ownerAddress = ownerAddress;
        _gameItemsContractInstance = GameItems(gameItemsContractAddress);
        isAdmin[_ownerAddress] = true;
    } 
    ...
    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
    ...
    function adjustAdminAccess(address adminAddress, bool access) external {
        require(msg.sender == _ownerAddress);
        isAdmin[adminAddress] = access;
    }  
```
## Recommended Mitigation Steps
Consider rechecking if this is intended behaviour or fixing to ensure a more seamless transition.
```
    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        isAdmin[_ownerAddress] = false;
        _ownerAddress = newOwnerAddress;
        isAdmin[newOwnerAddress] = true;
    }
```
***

***

# 3. Consider checking for max allowance in the `burnFrom` function before decreasing allowance

Lines of code* 

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L201

## Impact
The `burnFrom` function deducts the amount to burn from the spender's allowance before actually burning. This doesn't take into consideration that spenders that might have max approval. Such users, ideally shouldn't have their allowances decreased whenever they make perform transactions on behalf of the owner. 

```
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
## Recommended Mitigation Steps
Consider making a check for `type(uint256).max` approval before decreasing the allowance. Or making a call to the OZ ERC20 `_spendallowance` function instead.

***

# 4. `mint` function should use <= to account for last token when minting.

Lines of code* 

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L156
## Impact
The check against mint more than max supply uses the < operator, which ensures that the sum of totalSupply and amount to mint is always less than the max supply. Doing this will cause that the max supply will never be reached, leaving one last token unmintable. Consider using the <= operator to more appropriately cap the max supply.
```
require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply")
```
***


# 5. Introduce protections from approval race conditions

Lines of code* 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L171
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L184
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L163
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L375
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L250
## Impact
As a recommendation from the EIP20 standard, to prevent attack vectors like the one [described here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/)and [discussed here](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729), consider refactoring the approvals in such a way that they set the allowance first to 0 before setting it to another value for the same spender.
## Recommended Mitigation Steps
Consider approving to zero `first` to before granting new allowances.

***


# 6. Users can mint more fighters and potentially bypass max fighters allowed limit through `_safemint` reentrancy

Lines of code* 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L529

## Impact

When calling functions to mint a fighter in the FighterFarm, the internal `_createNewFighter` function is called. A check is made to ensure that user doesn't have more than the maximum allowed fighters. The issue here is that the `_safeMint` function that is called contains a callback function ` to.onERC721Received`. A user can reenter the function through the hook. 
```
   function _createNewFighter(
        address to, 
        uint256 dna, 
        string memory modelHash,
        string memory modelType, 
        uint8 fighterType,
        uint8 iconsType,
        uint256[2] memory customAttributes
    ) 
        private 
    {  
        require(balanceOf(to) < MAX_FIGHTERS_ALLOWED);
...
        _safeMint(to, newId);
        FighterOps.fighterCreatedEmitter(newId, weight, element, generation[fighterType]);
    }
```

Using the `mintFromMergingPool` function as an example,

- A winner(a contract) calls the `claimRewards` function.
- The function calls the `mintFromMergingPool` function, which calls the `createNewFunction` function.
- The `_createNewFighter`, which uses `_safeMint` internally.
- `_safeMint` calls onERC721Received() on the contract, the contract re-enters the function again, leading to multiple mints.

## Recommended Mitigation Steps
Consider using the reentrancy guard. 


# 7. Daily Game Item allowance limit can be bypassed through transfers.

Lines of code* 

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L291
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L169

## Impact
When creating a game item, a daily allowance limit is set for each user. When a user mints a game item, a check is made for the user's allowance upon which the quantity of game items minted is subtracted. 

```
    function mint(uint256 tokenId, uint256 quantity) external {
...
            if (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp) {
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
```
However, if a game item is transferrable, users can bypass the allowance limit by transferring the items between themselves. as it makes no check for user's allowances.  
```
function safeTransferFrom(
        address from, 
        address to, 
        uint256 tokenId,
        uint256 amount,
        bytes memory data
    ) 
        public 
        override(ERC1155)
    {
        require(allGameItemAttributes[tokenId].transferable);
        super.safeTransferFrom(from, to, tokenId, amount, data);
    }
```

## Recommended Mitigation Steps
Consider including a check for user's allowances upon transfer.
