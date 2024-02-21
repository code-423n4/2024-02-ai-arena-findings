# Missing check in `FighterFarm::redeemMintPass`

There is information that all input arrays should be the same length, but `iconsTypes` input is not checked in require statement.

```javascript
    function redeemMintPass(
        uint256[] calldata mintpassIdsToBurn,
        uint8[] calldata fighterTypes,
        uint8[] calldata iconsTypes,
        string[] calldata mintPassDnas,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) external {
        // q iconsTypes not included in the require statement
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && mintPassDnas.length == fighterTypes.length
                && fighterTypes.length == modelHashes.length && modelHashes.length == modelTypes.length
        );
.
.
.
```

# Double code in `AiArenaHelper` constructor

Constructor call function `addAtributeProbabilities(0, probabilities)`.

```javascript
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```
Then the same code is executed in the loop below:
```javascript
constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    }
```
Calling this function in constructor is unnecessary.