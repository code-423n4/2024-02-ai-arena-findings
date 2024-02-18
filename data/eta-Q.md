# Addressing Missing Fighter Information in the Notice of `FighterOps::viewFighterInfo`

## Description
The notice says `@notice Gets all of the relevant fighter information`, but it doesn't return information such as `id, iconsType, dendroidBool`, especially` iconsType, dendroidBool`, which are key variables related to the rarity of the fighter and are used by multiple functions in multiple contracts. Neglecting to include them in the return information could potentially hinder the functionality and interoperability of the codebase. Therefore, it's imperative to supplement the return information with these critical variables to ensure comprehensive data accessibility and seamless integration within the system.

It is suggested to provide complete return information as follows:

[FighterOps::Fighter](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L44C1-L45C27)
[FighterOps::viewFighterInfo](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterOps.sol#L78C1-L78C62)
```solidity
/// @notice Gets all of the relevant fighter information 
    function viewFighterInfo(
        Fighter storage self, 
        address owner
    ) 
        public 
        view 
        returns (
            address,
            uint256[6] memory,
            uint256,
            uint256,
            string memory,
            string memory,
            uint16,
+           uint8,
+           bool
        )
    {
        return (
            owner,
            getFighterAttributes(self),
            self.weight,
            self.element,
            self.modelHash,
            self.modelType,
            self.generation
+          self.iconsType
+          self.dendroidBool
        );
    }
```

## Recommendation

It's imperative to supplement the return information with these critical variables as shown in the part of Description

