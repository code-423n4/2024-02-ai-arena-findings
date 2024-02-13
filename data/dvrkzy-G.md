Unnecessary line of code in the constructor of `AiArenaHelper.sol`
===============

The following function is called in the constructor:

    function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }

And here is the constructor itself:

    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            //@audit- unnecessary line of code?
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
The code on `line:45` of the contract is useless as its already executed by the `addAttributeProbabilities` function which is called in the constructor.

The tests still pass if this line of code is removed:
`attributeProbabilities[0][attributes[i]] = probabilities[i];`