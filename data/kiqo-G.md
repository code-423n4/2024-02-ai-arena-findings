## Summary
In `VoltageManager.sol` changing the following mappings and event in the following fashion, leads to gas savings.

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/VoltageManager.sol#L36

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/VoltageManager.sol#L39

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/VoltageManager.sol#L16

```diff
- event VoltageRemaining(address spender, uint8 voltage);  
+ event VoltageRemaining(address spender, uint256 voltage);  

- mapping(address => uint32) public ownerVoltageReplenishTime;
+ mapping(address => uint256) public ownerVoltageReplenishTime;

- mapping(address => uint8) public ownerVoltage;
+ mapping(address => uint256) public ownerVoltage;

``` 

### POC
By making the changes and running 
`forge test --mt testUseVolatgeBattery --gas-report`
We get a gas saving of 130 gas when calling `VoltManager::useVoltageBattery`

And running 
`forge test --mt testSpendVoltageTriggerReplenishVoltage --gas-report`
We get a gas saving of 463 gas when calling `VoltManager::spendVoltage`