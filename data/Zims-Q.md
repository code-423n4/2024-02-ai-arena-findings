### QA Review
#### QA-1
Important functions lack emit events.
All contracts have a function to change the owner.
```javascript
    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        _ownerAddress = newOwnerAddress;
    }
```
Adding an event to emit that ownership has transferred is an essential part of such a function for several reasons:
- Transparency
- Security
- Interoperability

It would be beneficial for the project and make monitoring the contracts easier. Also a non-zero check is a good practice.
#### Recommendations
Add an event to emit that ownership has transferred and add a non-zero check.
```javascript
    function transferOwnership(address newOwnerAddress) external {
        require(msg.sender == _ownerAddress);
        
        require(newOwnerAddress != address(0), "Invalid address");
        emit OwnershipTransferred(_ownerAddress, newOwnerAddress);

        _ownerAddress = newOwnerAddress;
    }
```
PS. Many of the functions in the contracts lack emit events. It would be beneficial to add them for transparency and security.

#### QA-2
In `FightFarm::updateFighterStaking` there is a require statement that doesnt have a message.
```javascript
    function updateFighterStaking(uint256 tokenId, bool stakingStatus) external {
        require(hasStakerRole[msg.sender]);
        fighterStaked[tokenId] = stakingStatus;
        if (stakingStatus) {
            emit Locked(tokenId);
        } else {
            emit Unlocked(tokenId);
        }
    }
```
The whole codebase is consistent with the require statements having a message. This should be consistent.
```javascript
require(hasStakerRole[msg.sender], "Caller is not a staker");
```