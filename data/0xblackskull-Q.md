### `_setupRole` is deprecated in favor of `_grantRole` in `Neuron.sol`
- addMinter()
- addStaker()
- addSpender()

### No check for 0 or 1 in FighterFarm::incrementGeneration 
```
/// @param fighterType Type of fighter either 0 or 1 (champion or dendroid).
```

```solidity
function incrementGeneration(uint8 fighterType) external returns (uint8) {
        require(msg.sender == _ownerAddress);
        generation[fighterType] += 1;
        maxRerollsAllowed[fighterType] += 1;
        return generation[fighterType];
    }
```
But there is no check for fighterType would be 0 or 1, owner can trigger it by using any value. So add require check for fighterType is either 0 or 1
```
require(fighterType == 0 || fighterType == 1);
```

