# QA report

## L-01: Anyone can trigger `FighterCreated` event

### Summary

`https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L53`

This function is public and has no access control, anyone can call it without restriction so this could make tracking events quite annoying especially if combined with an implementation of The Graph in case this data is used somewhere. If not using The Graph there could still be a lot of events listed on explorers like Etherscan.io which could reduce readability.

### Mitigation

Consider adding access control or remove this function and emit the event using `emit FighterCreated(id, weight, element, generation)` as it would also be accessible wherever this function is imported now.

## L-02: Attributes for generation 0 are added twice in constructor resulting in unneccesary gas spending

### Summary

`https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L45`

In the constructor the function `addAttributeProbabilities` is called after which there is a for loop that does the same, except that the for loop also changes the `attributeToDnaDivisor` mapping. This results in spending extra gas which can be prevented.

### Mitigation

remove this function call

```diff
 constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
-        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```

## Info-01: No chainId used in signature could allow for transaction replays on other chains

### Summary

`https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L191`

In the signature message used to claim a fighter, there is no chainId variable included. In case the project decides to deploy on multiple sidechains this means that signatures can be reused to claim more fighters on another blockchain with the same signature.

### Mitigation

In order to mitigate this it's a good practice to include a chainId in the signed message.

## Info-02: No check on array sizes

### Summary

`https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139`

There is no check to see if the array sizes are the same length, it's considered to be best practice to check this in order to avoid problems later on.

## Info-03: All winners can choose customAttributes after winning a round

### Summary

`https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139`

In the event that a user wins a round, he is able to call the `claimRewards` function with the customAttributes of his choosing. I'm not sure if this is intentional or not so that's why I have added it info.