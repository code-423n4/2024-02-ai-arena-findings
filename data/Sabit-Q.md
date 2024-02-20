1. setAllowedBurningAddresses function can add one address multiple times

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/GameItems.sol#L185-L189

The issue is that setAllowedBurningAddresses can be called multiple times with the same address, resulting in duplicate entries in the allowedBurningAddresses mapping. This should be prevented by first checking if the address already exists before setting it.