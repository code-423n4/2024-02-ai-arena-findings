**Low** - There will always be **1** unused NRN token, because of the following statement in `mint` in `Neuron.sol`:

```
function mint(address to, uint256 amount) public virtual {
        
        //@audit this won't mint the last token of the MAX_SUPPLY
@>>>    require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```

**Dummy values**

**MAX_SUPPLY** = 100
**totalSupply** - 99

So when we try to call `mint` with `amount` = 1, the `require` will revert because
`99 + 1 < 100`. Fix the check the following way:

`require(totalSupply() + amount <= MAX_SUPPLY, "Trying to mint more than the max supply");
`

-----------------------------------------------------------------------------------

**Informational** - In `AiArenaHelper.sol` in the constructor, when we add attribute probabilities via `addAttributeProbabilities`, then when we iterate over the attributes, we again add the same probabilities to the same attributes. Remove the second adding of probabilities.

```
constructor(uint8[][] memory probabilities) {

        _ownerAddress = msg.sender;
        
        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;

        for (uint8 i = 0; i < attributesLength; i++) {
 
             //@audit this is not needed, remove it.
@>>>>        attributeProbabilities[0][attributes[i]] = probabilities[i];

            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];

        }
    } 
```

-----------------------------------------------------------------------------------

**Informational** - In `FighterFarm.sol`, `redeemMintPass` the natspec above the function says:

**"/// @dev This function requires the length of all input arrays to be equal."**.

However this is not the case, as `uint8[] calldata iconsTypes` is missed. Make sure to add the corresponding checks for it too.

```
function redeemMintPass(
        uint256[] calldata mintpassIdsToBurn,
        uint8[] calldata fighterTypes,
@>>>    uint8[] calldata iconsTypes,
        string[] calldata mintPassDnas,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        
@>>>    //@audit no checks for iconsTypes
        require(
            mintpassIdsToBurn.length == mintPassDnas.length && 
            mintPassDnas.length == fighterTypes.length && 
            fighterTypes.length == modelHashes.length &&
            modelHashes.length == modelTypes.length
        );

        for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {

            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));

            _mintpassInstance.burn(mintpassIdsToBurn[i]);

            _createNewFighter(
                msg.sender, 
                uint256(keccak256(abi.encode(mintPassDnas[i]))), 
                modelHashes[i], 
                modelTypes[i],
                fighterTypes[i],
                iconsTypes[i],
                [uint256(100), uint256(100)]
            );
        }
    }
```

-----------------------------------------------------------------------------------


**Informational** - In `Neuron.sol` you can remove the following `require` in `burnFrom` and `claim`:

```
require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
```

The require in both functions is not needed, because in `claim`, `transferFrom` is used which already ensures that user has the needed allowance to transfer from the `treasuryAddress` in this example. The same goes for `burnFrom` but there `      uint256 decreasedAllowance = allowance(account, msg.sender) - amount;` will revert if the user doesn't have the needed allowance, so the require statement there is useless too.

-----------------------------------------------------------------------------------
