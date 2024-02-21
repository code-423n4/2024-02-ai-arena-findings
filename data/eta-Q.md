# [L-01]  `ownerVoltage[msg.sender] = 100` causes losses of users’ original balance in the `VoltageManager::useVoltageBattery`

## Impact
The users’ remaining voltage is updated to `ownerVoltage[msg.sender] = 100`, instead of `ownerVoltage[msg.sender] += 100`. It directly results in the losses of users’ original balance, potentially leading to financial losses for users. 

## Proof of Concept
In the function `VoltageManager::useVoltageBattery`, after the user uses a voltage battery to replenish voltage, the user's remaining voltage is updated to `ownerVoltage[msg.sender] = 100`, instead of `ownerVoltage[msg.sender] += 100`. This would cause the user to lose their original balance.

For instance, if the user's original balance is `ownerVoltage[msg.sender] == 99`,

1)The user calls the `useVoltageBattery()` function, and the user's voltage balance is updated to `ownerVoltage[msg.sender] = 100`.
2)However, the user's actual voltage balance should be `ownerVoltage[msg.sender] = 199`, calculated as 99 + 100 = 199, resulting in a direct loss of 99. This would lead to financial loss for the user and undermine the platform's credibility.

Therefore, it is advisable to modify `ownerVoltage[msg.sender] = 100` to `ownerVoltage[msg.sender] += 100`, as depicted below:

[VoltageManager::useVoltageBattery](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L97C1-L97C40)
```solidity
    function useVoltageBattery() public {
        require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
-       ownerVoltage[msg.sender] = 100;
+       ownerVoltage[msg.sender] += 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }
```

## Tools Used
Manual Review

## Recommendation

It is advisable to modify `ownerVoltage[msg.sender] = 100` to `ownerVoltage[msg.sender] += 100`, as shown in the part of Proof of Concept:

```solidity
    function useVoltageBattery() public {
        require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
-       ownerVoltage[msg.sender] = 100;
+       ownerVoltage[msg.sender] += 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }
```




# [L-02] Addressing Missing Fighter Information in the Notice of `FighterOps::viewFighterInfo`

## Impact
The notice advertised as returning "all of the relevant fighter information" is currently missing key variables: `id`, `iconsType`, and `dendroidBool` in the `FighterOps::viewFighterInfo`. These variables are crucial for determining the rarity of fighters and are used extensively across multiple contracts and functions.

## Proof of Concept
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

## Tools Used
Manual Review

## Recommendation

It's imperative to supplement the return information with these critical variables as shown in the part of Description

```solidity
+          self.iconsType
+          self.dendroidBool

```