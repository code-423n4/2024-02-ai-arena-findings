## [L-01] Lack of access control

fighterCreatedEmitter() is public, allowing anyone to arbitrarily modify the data storing the new Fighter NFT.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L53
```
   /// @notice Emits a FighterCreated event.
    function fighterCreatedEmitter(
        uint256 id,
        uint256 weight,
        uint256 element,
        uint8 generation
    ) 
        public 
    {
        emit FighterCreated(id, weight, element, generation);
    }
```


## [L-02]Potential Voltage loss
There are some senarios:
1. Users can replenish Voltage every day, but _replenishVoltage is executed within spendVoltage().
 The spendVoltage() function takes the voltageSpent parameter as input. 
Users who only want to replenish Voltage but lack of awareness or accidental input of voltageSpent will lose Voltage.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L110
```
ownerVoltage[spender] -= voltageSpent;
```
It is recommended to separate replenishVoltage into a standalone function.

2. The useVoltageBattery() only checks if the user's voltage is less than 100. Users with non-zero voltage will lose their remaining Voltage.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L94
```
require(ownerVoltage[msg.sender] < 100);
```

## [L-03] URI returns false data.

The owner can add admin access for a user through adjustAdminAccess().
Multiple users may have admin access.
setTokenURI() and createGameItem() are called by a user with admin access.
After createGameItem(), other users with admin access can freely modify the tokenURI using setTokenURI.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L194
```
function setTokenURI(uint256 tokenId, string memory _tokenURI) public {
        require(isAdmin[msg.sender]);
        _tokenURIs[tokenId] = _tokenURI;
    }
```
The value of the NFTs depends on their metadata which includes critical information such as traits, rarity, images, etc.
Additionally, they can freely set non-existent tokenID and _tokenURI.
Bypass the check for uri() 
```if (bytes(customURI).length > 0)```.
Return non-existing tokenID along with its fake metadata.
This may mislead users. 

## [L-04] Use transferFrom may loss NFT

When using the transferFrom function of an ERC721 contract to send an NFT, if the receiving address is a smart contract and does not support ERC721, the NFT can be permanently lost.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L338
```
 function transferFrom(
        address from, 
        address to, 
        uint256 tokenId
    ) 
        public 
        override(ERC721, IERC721)
    {
        require(_ableToTransfer(tokenId, to));
        _transfer(from, to, tokenId);
    }
```
Recommendation:
Ensure receiving contract have  onERC721Received method.


## [L-05] Precision  loss
bpsLostPerLoss is [adjusted by the msg.sender](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L226) with isAdmin role, and there are no upper or lower limits.
When calculating curStakeAtRisk, possible precision loss.

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L439
```
curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
```
Causing users unable to [reclaim](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L461) stake-at-risk puts the NRN back into their staking pool.
