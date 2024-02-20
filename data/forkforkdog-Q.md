### `_delegatedAddress` is not changeable

There is no setter function for FighterFarm.sol:_delegatedAddress besides constructor. 

_delegatedAddress is used as a gatekeeper for crucial minting function, and redeploying contract for address change is not an option without breaking whole game. 

```
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
        require(verify(msgHash, signature, _delegatedAddress));

...
}
```

Consider implementing same fuction from AAMintPass.sol
```
function setDelegatedAddress(address _delegatedAddress) external {
        require(msg.sender == founderAddress);
        delegatedAddress = _delegatedAddress;
    }
```