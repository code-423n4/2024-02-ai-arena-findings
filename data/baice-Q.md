Add return value checks when user claim neuron token successfully
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L143

```solidity
   function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
@-      transferFrom(treasuryAddress, msg.sender, amount);

@+      require(transferFrom(treasuryAddress, msg.sender, amount),"ERC20: claim failed");        
        emit TokensClaimed(msg.sender, amount);
    }
```

