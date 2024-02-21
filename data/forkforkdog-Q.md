### RankedBattle.sol:setBpsLostPerLoss() mid-round change could be unfair for users with stake losses


[RankedBattle.sol:bpsLostPerLoss()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L226) function is designed to specify the percentage points of a player's staked amount that will be deducted in the event of a battle loss.


In RankedBattle.sol, the bpsLostPerLoss() function lacks execution control, allowing it to be executed unrestrictedly at any point within the round's timeframe. Such unregulated execution may lead to rule alterations mid-round, potentially disadvantaging users who have already incurred stake losses under the initially established rules.


```
    function setBpsLostPerLoss(uint256 bpsLostPerLoss_) external {
        require(isAdmin[msg.sender]);
        bpsLostPerLoss = bpsLostPerLoss_;
    }
```

Would recommend implementing a constraint on setBpsLostPerLoss() to allow its execution only if no player has participated in the current round, akin to the limitations imposed on setNewRound().


```
    function setBpsLostPerLoss(uint256 bpsLostPerLoss_) external {
        require(isAdmin[msg.sender]);
+		 require(totalAccumulatedPoints[roundId] > 0);
        bpsLostPerLoss = bpsLostPerLoss_;
    }
```

### RankedBattle.sol:stakeNRN Should revert on non-zero message value

User can accidentally send non-zero value in attempt to stakeNRN, and there is no way to rescue ether form Ranked battle contract which leads to permanent lost of ether for the user. 

```
   function stakeNRN(uint256 amount, uint256 tokenId) external {
        require(amount > 0, "Amount cannot be 0");
        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
        require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance");
        require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");

        _neuronInstance.approveStaker(msg.sender, address(this), amount);
        bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
        if (success) {
            if (amountStaked[tokenId] == 0) {
                _fighterFarmInstance.updateFighterStaking(tokenId, true);
            }
            amountStaked[tokenId] += amount;
            globalStakedAmount += amount;
            stakingFactor[tokenId] = _getStakingFactor(tokenId, _stakeAtRiskInstance.getStakeAtRisk(tokenId));
            _calculatedStakingFactor[tokenId][roundId] = true;
            emit Staked(msg.sender, amount);
        }
    }
    
```

Consider using require statement to restrict non-zero message value. 


```
   function stakeNRN(uint256 amount, uint256 tokenId) external {
   +    require(msg.value == 0, "No ether please");
        require(amount > 0, "Amount cannot be 0");
        require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
        require(_neuronInstance.balanceOf(msg.sender) >= amount, "Stake amount exceeds balance");
        require(hasUnstaked[tokenId][roundId] == false, "Cannot add stake after unstaking this round");

        _neuronInstance.approveStaker(msg.sender, address(this), amount);
        bool success = _neuronInstance.transferFrom(msg.sender, address(this), amount);
        if (success) {
            if (amountStaked[tokenId] == 0) {
                _fighterFarmInstance.updateFighterStaking(tokenId, true);
            }
            amountStaked[tokenId] += amount;
            globalStakedAmount += amount;
            stakingFactor[tokenId] = _getStakingFactor(tokenId, _stakeAtRiskInstance.getStakeAtRisk(tokenId));
            _calculatedStakingFactor[tokenId][roundId] = true;
            emit Staked(msg.sender, amount);
        }
    }
    
```

### `_delegatedAddress` is not changeable

There is no setter function for FighterFarm.sol:_delegatedAddress besides constructor. 

_delegatedAddress is used as a gatekeeper for crucial minting function, and redeploying contract for address change is not an option without breaking whole game. 

```
function claimFighters(
        uint8[2] calldata numToMint,
        bytes calldata signature,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        bytes32 msgHash = bytes32(keccak256(abi.encode(
            msg.sender, 
            numToMint[0], 
            numToMint[1],
            nftsClaimed[msg.sender][0],
            nftsClaimed[msg.sender][1]
        )));
        require(verify(msgHash, signature, _delegatedAddress));

...
}
```

Consider implementing same fuction from AAMintPass.sol
```
function setDelegatedAddress(address _delegatedAddress) external {
        require(msg.sender == founderAddress);
        delegatedAddress = _delegatedAddress;
    }
```
