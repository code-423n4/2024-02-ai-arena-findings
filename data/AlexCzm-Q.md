### L-01 `attributeProbabilities` are set twice in AiArenaHelper constructor;
Remove `addAttributeProbabilities()` [function call](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L45) from `AiArenaHelper`  constructor. 

```solidity
    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);// @audit-qa remove this function call

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];// because you are setting attr prob here
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```

### L-02 `MINTER_ROLE` can mint back any burned NRN. 
Neuron contract [expose](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L163-L165) externally the `burn` function, allowing any NRN holder to destroy tokens they hold. The maximum total supply should be decreased by the amount of burned tokens. 
Additionally, in `mint` function use `<=` instead `<` to better reflect what `MAX_SUPPLY` means. 

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L156

### L-03 Once staked, a fighter is considered staked even if its `amountStaked` balance == 0. 
To allow transferring a previously staked fighter, a player must call `unstakeNRN` even if `amountStaked` for that fighter is 0. 
Consider improving the player UX and read the `stakedBalance` directly. 

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L286

### L-04 Create an Enum with all possible battle results
Currently magic numbers 0, 1, and 2 are used as battle results. 
Consider creating and using an enum instead. Moreover avoid using value 0 as a valid battle results:
```solidity
    enum battleResult {
        invalidResult,
        won,
        tie,
        lost
    }

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L440

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L505-L513

### L-05 Allows only voltage spender to consume energy
Currently any addresses can call `spendVoltage` to reduce its energy. 
Even if unlikely, this can open the door for abuses. Consider keeping only selected addresses to spend voltage 
```solidity

    function spendVoltage(address spender, uint8 voltageSpent) public {
-        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
+        require(allowedVoltageSpenders[msg.sender]);
...
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L106

### L-06 Players can get an advantage and mint game items with daily allowances/  with multiple addresses 
GameItems contract implements a few restrictions related to the amount of items can be minted daily per address. Players can get an advantage by minting items with daily allowance. Moreover, there is an `transferable` item options.
Consider  to allow only fighter owners to mint items which can't be transferred  or which have an daily allowance.  

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L224

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L312-L314

### L-07 Missing length check
`iconsTypes` array length check is missing but all other arrays length is checked. 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L233-L248


### NC-01 Missing @param natspec

- `iconsTypes` from `RedeemMintPass`
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L236

 - `generation` from `createPhysicalAttributes`
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L85
