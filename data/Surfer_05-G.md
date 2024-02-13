File name: src/AiArenaHelper.sol

The constructor can be optimised. We are initialising the attributeProbabilities. Firstly calling the function and then manually. 
```solidity
constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
@==>    addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
@==>        attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```
The gas cost for this contract is as follows :

src/AiArenaHelper.sol:AiArenaHelper contract |                 |        |        |        |         |
|----------------------------------------------|-----------------|--------|--|--------|---------|
| Deployment Cost                              | Deployment Size |        |        |        |         |
| 1741279                                      | 8445            |   

Now, after reducing the initialisation to once, the code should be : 
```
constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        // commenting out the next line as not needed
@==>    // addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    }      
```
And now the cost will be as follows : 
src/AiArenaHelper.sol:AiArenaHelper contract |                 |        |        |        |         |
|----------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                              | Deployment Size |        |        |        |         |
| 1660491                                      | 8129 

Thus, we reduce the cost by 
```
1741279 - 1660491 = 80788 gas
``` 