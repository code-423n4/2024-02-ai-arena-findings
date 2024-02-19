## Duplicate code exists in AiArenaHelper.sol, resulting in unnecessary waste of gas

## Impact
In the constructor of the AiArenaHelper contract, the statement `addAttributeProbabilities(0, probabilities);` calls the addAttributeProbabilities function for initialization, but this statement has the same function as the statement in the for loop, resulting in unnecessary waste of gas.

## Proof of Concept 

```
constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities); //here！！！

        uint256 attributesLength = attributes.length; 
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i]; // When i equals 0, the function here is repeated！
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    }
```

```
 function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```

### Tools Used
Manual review

## Recommended Mitigation Steps
1. Delete `addAttributeProbabilities(0, probabilities);`
2. Add the check `require(probabilities.length == 6, "Invalid number of attribute arrays");` in `addAttributeProbabilities`