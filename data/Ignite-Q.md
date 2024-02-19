# [Low-01] - Unsigned Integer Overflow

## Impact

This function calculates the total number of fighters to mint by summing up the quantities of two types of fighters specified in the `numToMint` array at line 207.

```solidity=191
function claimFighters(
    uint8[2] calldata numToMint,
    bytes calldata signature,
    string[] calldata modelHashes,
    string[] calldata modelTypes
) 
    external 
{
    bytes32 msgHash = bytes32(keccak256(abi.encode(
        msg.sender, 
        numToMint[0], 
        numToMint[1],
        nftsClaimed[msg.sender][0],
        nftsClaimed[msg.sender][1]
    )));
    require(Verification.verify(msgHash, signature, _delegatedAddress));
    uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
    require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
    nftsClaimed[msg.sender][0] += numToMint[0];
    nftsClaimed[msg.sender][1] += numToMint[1];
    for (uint16 i = 0; i < totalToMint; i++) {
        _createNewFighter(
            msg.sender, 
            uint256(keccak256(abi.encode(msg.sender, fighters.length))),
            modelHashes[i], 
            modelTypes[i],
            i < numToMint[0] ? 0 : 1,
            0,
            [uint256(100), uint256(100)]
        );
    }
}
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L191-L222

However, if `numToMint[0]` and `numToMint[1]` are summed up to a value greater than 255, this function will revert due to overflow.

```solidity=207
uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L207

This line will sum up `numToMint[0]` and `numToMint[1]` before converting to `uint16`. Since both `numToMint[0]` and `numToMint[1]` are of type uint8, and the maximum value for a uint8 is 255, an overflow will occur if their sum exceeds this value.



## Proof of Concept

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L207

## Tools Used

Manual Review

## Recommended Mitigation Steps

Convert `numToMint` to uint16 type before summing it up.

For example:

```diff
function claimFighters(
    uint8[2] calldata numToMint,
    bytes calldata signature,
    string[] calldata modelHashes,
    string[] calldata modelTypes
) 
    external 
{
    bytes32 msgHash = bytes32(keccak256(abi.encode(
        msg.sender, 
        numToMint[0], 
        numToMint[1],
        nftsClaimed[msg.sender][0],
        nftsClaimed[msg.sender][1]
    )));
    require(Verification.verify(msgHash, signature, _delegatedAddress));
-    uint16 totalToMint = uint16(numToMint[0] + numToMint[1]);
+    uint16 totalToMint = uint16(numToMint[0]) + uint16(numToMint[1]);
    require(modelHashes.length == totalToMint && modelTypes.length == totalToMint);
    nftsClaimed[msg.sender][0] += numToMint[0];
    nftsClaimed[msg.sender][1] += numToMint[1];
    for (uint16 i = 0; i < totalToMint; i++) {
        _createNewFighter(
            msg.sender, 
            uint256(keccak256(abi.encode(msg.sender, fighters.length))),
            modelHashes[i], 
            modelTypes[i],
            i < numToMint[0] ? 0 : 1,
            0,
            [uint256(100), uint256(100)]
        );
    }
}
```