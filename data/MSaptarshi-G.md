## G-01 Loading from the memory cost less gas than loading from the storage 

1.https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L130
2.https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L196

### Recommendation: Load from the memory 
1.
```
+ uint256 recipientsLength = recipients.length;  
 - require(recipients.length == amounts.length);
  + require(recipientsLength == amounts.length);
- uint256 recipientsLength = recipients.length;
   for (uint32 i = 0; i < recipientsLength; i++) 
```

2.
```
+ uint256 getAllowance = allowance(account, msg.sender) ;
+       require(getAllowance >= amount, "ERC20: burn amount exceeds allowance");
-        require( allowance(account, msg.sender) >= amount, 
                        "ERC20: burn amount exceeds allowance");
+       uint256 decreasedAllowance = getAllowance - amount;
-       uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
```
