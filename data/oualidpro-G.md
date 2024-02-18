## Summary
## Gas Optimizations


| |Issue|Instances|Gas Saved|
|-|:-|:-:|-:|
| [GAS-1](#GAS-1) | Avoid external call for event emiting to save gas | 1 | 3301 |
| [GAS-2](#GAS-2) | Remove redendente code to save gas | 1 | - |
| [GAS-3](#GAS-3) | Optimize Deployment Size by Fine-tuning IPFS Hash (New Instances) | 1 | 10600 |
| [GAS-4](#GAS-4) | Use assembly to emit events (New Instances) | 1 | 38 |
| [GAS-5](#GAS-5) | Using bools for storage incurs overhead (New Instances) | 1 | 100 |
| [GAS-6](#GAS-6) | Optimize names to save gas (New Instances) | 1 | 22 |
| [GAS-7](#GAS-7) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead (New Instances) | 1 | - |
| [GAS-8](#GAS-8) | State variable read in a loop (New Instances) | 1 | 97 |
### <a name="GAS-1"></a>[GAS-1] Avoid external call for event emiting to save gas
The `_createNewFighter` internal function perform an external call to `fighterCreatedEmitter()` in the `FighterOps` contract to simply emit an event.
Therefore, to save gas you should copy that event into the `FighterFarm` and call it internaly to save `3301 gas`

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: /src/FighterFarm.sol

function _createNewFighter(
        address to, 
        uint256 dna, 
        string memory modelHash,
        string memory modelType, 
        uint8 fighterType,
        uint8 iconsType,
        uint256[2] memory customAttributes
    ) 
        private 
    {  
        require(balanceOf(to) < MAX_FIGHTERS_ALLOWED);
        uint256 element; 
        uint256 weight;
        uint256 newDna;
        if (customAttributes[0] == 100) {
            (element, weight, newDna) = _createFighterBase(dna, fighterType);
        }
        else {//@audit-info require more investigations (it seems like we can predict what kind of Fighter we can get in minting)
            element = customAttributes[0];
            weight = customAttributes[1];
            newDna = dna;
        }
        uint256 newId = fighters.length;

        bool dendroidBool = fighterType == 1;
        FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(
            newDna,
            generation[fighterType],
            iconsType,
            dendroidBool
        );
        fighters.push(
            FighterOps.Fighter(
                weight,
                element,
                attrs,
                newId,
                modelHash,
                modelType,
                generation[fighterType],
                iconsType,
                dendroidBool
            )
        );
        _safeMint(to, newId);
        FighterOps.fighterCreatedEmitter(newId, weight, element, generation[fighterType]); //@audit GAS: you can copy the same event and call it directly in this function to
    }

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L484-L531)

</details>

### <a name="GAS-2"></a>[GAS-2] Remove redendente code to save gas
The value of `attributeProbabilities` is calculated two times when the contract is deployed for the first time in the constractor function.
the first time when the `addAttributeProbabilities()` funtion is called, and the second time throught the `for` loop.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/AiArenaHelper.sol

constructor(uint8[][] memory probabilities) {
    _ownerAddress = msg.sender;

    // Initialize the probabilities for each attribute
    addAttributeProbabilities(0, probabilities);

    uint256 attributesLength = attributes.length;
    for (uint8 i = 0; i < attributesLength; i++) {
        attributeProbabilities[0][attributes[i]] = probabilities[i]; // @audit Gas: this line of code could be removed as it is already calculated before
        attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
    }
} 

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L41-L52)

</details>

### <a name="GAS-3"></a>[GAS-3] Optimize Deployment Size by Fine-tuning IPFS Hash
Optimizing the deployment size of a smart contract is vital to minimize gas costs, and one way to achieve this is by fine-tuning the IPFS hash appended by the Solidity compiler as metadata. This metadata, consisting of 53 bytes, increases the gas required for contract deployment by approximately 10,600 gas due to bytecode costs, and additionally, up to 848 gas due to calldata costs, depending on the proportion of zero and non-zero bytes. Utilize the --no-cbor-metadata compiler flag to prevent the compiler from appending metadata. However, this approach has a drawback as it can complicate the contract verification process on block explorers like Etherscan, potentially reducing transparency.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

7: library FighterOps {

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L7)

</details>

### <a name="GAS-4"></a>[GAS-4] Use assembly to emit events
We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

61:         emit FighterCreated(id, weight, element, generation);

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L61)

</details>

### <a name="GAS-5"></a>[GAS-5] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from 'false' to 'true', after having been 'true' in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

//@audit change `dendroidBool` to `uint256`
36:     struct Fighter {
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
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L36-L46)

```solidity
File: src/FighterOps.sol

//@audit change `dendroidBool` to `uint256`
36:     struct Fighter {
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
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L36-L46)

```solidity
File: src/GameItems.sol

//@audit change `finiteSupply` to `uint256`
//@audit change `transferable` to `uint256`
35:     struct GameItemAttributes {
            string name;
            bool finiteSupply;
            bool transferable;
            uint256 itemsRemaining;
            uint256 itemPrice;
            uint256 dailyAllowance;
        }  

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L35)

</details>

### <a name="GAS-6"></a>[GAS-6] Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save **128 gas** each during deployment, and renaming functions to have lower method IDs will save **22 gas** per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

// @audit fighterCreatedEmitter(uint256,uint256,uint256,uint8) ==> fighterCreatedEmitter_M1y(uint256,uint256,uint256,uint8),00004d48
7: library FighterOps {

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L7)

</details>

### <a name="GAS-7"></a>[GAS-7] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead
> When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra [** 22 - 28 gas **](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/FighterOps.sol

//@audit `generation` is `uint8`
57:         uint8 generation

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterOps.sol#L57)

</details>

### <a name="GAS-8"></a>[GAS-8] State variable read in a loop (New Instances)
The state variable should be cached in a local variable rather than reading it on every iteration of the for-loop, which will replace each Gwarmaccess (**100 gas**) with a much cheaper stack read.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/MergingPool.sol

//@audit `roundId` is read in this loop
149:         for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {

```
[Link to code](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L149)

</details>
