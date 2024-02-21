### L1: keccak256 already returns bytes32 datatype
**Description :**

```bytes32 msgHash = bytes32(keccak256(abi.encode(``` 

Here In [L199](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L199) claimFighters() function, keccak256 already returns bytes32 and don't need to typecast to bytes32 again.

**Recommendation :** avoid typecasting again to bytes32.

### L2: apply check to ensure lengths of input parameters of claimRewards() to be equal. 
**Description :**

```function claimRewards(```
```string[] calldata modelURIs,```
```string[] calldata modelTypes,```
```uint256[2][] calldata customAttributes```
```) ```

Here in [L139-L143](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L139-L143) claimRewards() function, check should be made to make sure lengths of modelURIs, modelTypes and customAttributes are equal.

**Recommendation :**

```require(```
```modelURIs.length == modelTypes.length &&```
```modelTypes.length == customAttributes.length,```
```"Lengths of modelURIs, modelTypes, and customAttributes must be equal"```
```);```

### L3 : getFighterPoints() retrieves point of only first tokenId.
**Description :**

```function getFighterPoints(uint256 maxId) public view returns(uint256[] memory) ```
```{```
       ``` uint256[] memory points = new uint256[](1);```
        ```for (uint256 i = 0; i < maxId; i++) {```
         ```points[i] = fighterPoints[i];```
        ```}```
       ``` return points;```
   ``` } ```

Here in [L205-L211](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L205-L211) getFighterPoints function, the length of the points array is set to 1, regardless of the value of maxId. Therefore, whatever be the size of maxId, the function will still return an array of length 1.so it will retrieve point of only one tokenId.

Recommendation : 

initialize the points array with the correct length.
```uint256[] memory points = new uint256[](maxId+1);```