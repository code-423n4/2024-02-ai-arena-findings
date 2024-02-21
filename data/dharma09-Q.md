## Insecure Elliptic Curve Recovery Mechanism which may permit signature replays


In `FighterFarm.sol` contract implemented claimFighters function and signature verification. For that function call use external call. In another function `verify` which is vulnerable to signature mallebility.this function doesn't check for s is less than token value.



```solidity
File: src/FighterFarm.sol
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
        require(Verification.verify(msgHash, signature, _delegatedAddress)); //@audit 
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
The ecrecover() function is a low-level cryptographic function that should be utilized after appropriate sanitizations have been enforced on its arguments, namely on the s and v values. This is due to the inherent trait of the curve to be symmetrical on the x-axis and thus permitting signatures to be replayed with the same x value (r) but a different y value (s).

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Verification.sol
```solidity
File: src/Verification.sol
function verify(
        bytes32 msgHash, 
        bytes memory signature,
        address signer
    ) 
        public 
        pure 
        returns(bool) 
    {
        require(signature.length == 65, "invalid signature length");

        bytes32 r;
        bytes32 s;
        uint8 v;

        assembly {
            /*
            First 32 bytes stores the length of the signature

            add(sig, 32) = pointer of sig + 32
            effectively, skips first 32 bytes of signature

            mload(p) loads next 32 bytes starting at the memory address p into memory
            */

            // first 32 bytes, after the length prefix
            r := mload(add(signature, 32))
            // second 32 bytes
            s := mload(add(signature, 64))
            // final byte (first byte of the next 32 bytes)
            v := byte(0, mload(add(signature, 96)))
        }
        bytes memory prefix = "\x19Ethereum Signed Message:\n32";
        bytes32 prefixedHash = keccak256(abi.encodePacked(prefix, msgHash));
@>        return ecrecover(prefixedHash, v, r, s) == signer;
    }
```
- [FighterFarm.sol#L206](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L206)
### Proof of Concept
Here's a simple proof of concept demonstrating this vulnerability.
```solidity
pragma solidity ^0.8.0;
contract Test {
    function testRecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) public pure returns (address) {
        return ecrecover(hash, v, r, s);
    }
}
```
In this contract, the testRecover() function takes a hash and a signature as parameters and returns the address derived from the signature. An attacker could call this function with a different signature that corresponds to the same hash, effectively impersonating the original signer.

It's important to note that this vulnerability can lead to serious security issues, especially in scenarios where signatures are used for authentication or identification purposes, such as in decentralized identity systems, non-fungible tokens (NFTs), multi-signature wallets, and decentralized finance (DeFi) applications.

### Recommended Mitigation Steps
I advise them to be sanitized by ensuring that v is equal to either 27 or 28 (v ∈ {27, 28}) and to ensure that s is existent in the lower half order of the elliptic curve (0 < s < secp256k1n ÷ 2 + 1) by ensuring it is less than 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A1.
A reference implementation of those checks can be observed in the ECDSA library of OpenZeppelin and the rationale behind those restrictions exists within Appendix F of the Yellow Paper.

Alternatively, use the latest EIP-712 standard, which provides a more secure way of handling signatures in Solidity. This standard includes a structured data argument that can be hashed and signed, providing better protection against replay attacks and signature malleability.
