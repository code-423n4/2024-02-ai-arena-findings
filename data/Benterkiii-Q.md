# Public functions that hasn't used internally can be external

Instances (4)

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AAMintPass.sol#L145
```solidity
    function totalSupply() public view returns (uint256) {
        return numTokensOutstanding;
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AAMintPass.sol#L150
```solidity
    function contractURI() public pure returns (string memory) {
        return "ipfs://bafybeifdvzwsjpxrkbyhqpz3y57j7nwm3eyyc3feqppqggslea3g5kk3jq";
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L53
```solidity
    function fighterCreatedEmitter(
```
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L79
```solidity
    function viewFighterInfo(
```