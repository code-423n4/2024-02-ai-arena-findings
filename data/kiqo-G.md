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


## Gas Saving in `FighterOps.sol`
In [`FighterOps.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol) , [`FighterPhysicalAttributes`](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterOps.sol#L26) are defined as
``` 
    struct FighterPhysicalAttributes {
        uint256 head;
        uint256 eyes;
        uint256 mouth;
        uint256 body;
        uint256 hands;
        uint256 feet;
    }
``` 
This structure is stored in [`FighterFarm.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L69), and occupies 6 storage slots, however the values that are set for these attributes in [`AiArenaHelper::dnaToIndex`](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/AiArenaHelper.sol#L169C14-L169C24) can only have a maximum value of 256.
Therefore we can change it to store them all in a single storage slot

``` 
 struct FighterPhysicalAttributes {
        uint16 head;
        uint16 eyes;
        uint16 mouth;
        uint16 body;
        uint16 hands;
        uint16 feet;
    }
```

Also for `Fighter`in the same file
```
 struct Fighter {
        uint256 weight;
        uint256 element;
        FighterPhysicalAttributes physicalAttributes;
        uint256 id;
        string modelHash;
        string modelType;
        uint8 generation;
        uint8 iconsType;
        bool dendroidBool;
    }
```
`Weight` and `element` fields that are set in [`FighterFarm::_createFighterBase](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/FighterFarm.sol#L462) can only have values below 256 therefore the struct can be changed to 
```
struct Fighter {
        uint8 weight;
        uint8 element;
        uint8 generation;
        uint8 iconsType;
        bool dendroidBool;
        FighterPhysicalAttributes physicalAttributes;
        uint256 id;
        string modelHash;
        string modelType;
      
    }
```
For further gas savings in storage