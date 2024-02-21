## 1. Non-Critical-01. Using dynamic arrays

### Relevant GitHub Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L17

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L20

## Summary

Dynamic arrays are used instead of fixed size arrays

## Vulnerability Details

Use dynamic arrays to store elements, although there are no plans to add new arrays in the future.

## Impact

Using fixed-size arrays can help avoid potential problems associated with unexpected growth of dynamic structures. They are easy to manage in memory because their size remains constant throughout execution. Using fixed-size arrays saves gas

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

I recommend specifying a fixed size for arrays that will not grow over time.

```diff

- string[] public attributes = [
  "head",
  "eyes",
  "mouth",
  "body",
  "hands",
  "feet"
  ];


+ string[6] public attributes = [
"head",
"eyes",
"mouth",
"body",
"hands",
"feet"
];
```

---

## 2. Non-Critical-02. Two identical actions in the constructor

### Relevant GitHub Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41-L52

## Summary

There are two identical actions in the constructor.

## Vulnerability Details

In the constructor there is a function `addAttributeProbabilities(0, probabilities);` which is used to Add attribute probabilities for a given generation. But also in this `constructor` below this function there is a loop that also Add attribute probabilities for generation 0

- loop in the constructor:

```solidity
for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
```

- function `addAttributeProbabilities`

```solidity
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

You should consider removing the `addAttributeProbabilities(0, probabilities);` function in the constructor, or removing the add attributes in the loop

---

## 3. Non-Critical-03. Incorrect check of require() condition

### Relevant GitHub Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131-L139

## Summary

The function `addAttributeProbabilities()` has a condition `require(probabilities.length == 6,"Invalid number of attribute arrays");`

## Vulnerability Details

In this statement, the array `probabilities` must always have a length equal to **6** in order to assign values to **attributes** that are also equal to **6**. The attributes array is dynamic, so it can be assigned an additional value or deleted.

## Impact

In the future, if the attributes array changes, the entire system will be broken when this function is used.

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

- You should consider comparing not with **6** but with the `attributes.length`
- Also, if the array of `attributes` will be unchanged, I recommend you to consider the option of introducing a `uint8 constant len = 6` and use it in loops, instead of setting a variable that will be equal to `attributes.length` every time.

---

## 4. Non-critical-04. Getter

### Relevant GitHub Links

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L279-L281
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L285-L287

## Summary

Missing from `get` function names

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

For better understanding and for code readability, you should add `get` to the function names

---

## 5. Non-Critical-05. No events

### Relevant GitHub Links

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L93-L96
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L101-L104
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L109-L112

## Summary

These functions have no `events`

## Vulnerability Details

Users need to know which address is `Minter`, `Spender`, `Staker`

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

It is recommended to add `events` to the end of each function

---

## 6. Non-critical-06. Strict condition

### Relevant GitHub Links

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L155-L159

## Summary

function has too strict a condition

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

It's worth looking at the condition and making it more permissible.

```diff
function mint(address to, uint256 amount) public virtual {
        require(
-            totalSupply() + amount < MAX_SUPPLY,
            "Trying to mint more than the max supply"
        );
        require(
            hasRole(MINTER_ROLE, msg.sender),
            "ERC20: must have minter role to mint"
        );
        _mint(to, amount);
    }
```

```diff
function mint(address to, uint256 amount) public virtual {
        require(
+            totalSupply() + amount <= MAX_SUPPLY,
            "Trying to mint more than the max supply"
        );
        require(
            hasRole(MINTER_ROLE, msg.sender),
            "ERC20: must have minter role to mint"
        );
        _mint(to, amount);
    }
```

## 7. Low-01. No nested array length check

### Relevant GitHub Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L131-L139

## Vulnerability Details

The `addAttributeProbabilities()` function accepts two parameters

- `uint256 generation`
- `uint8[][] memory probabilities`

This function has a check on the length of the `probabilities` array, but there is no check on the length of nested arrays.

## Impact

If by chance the `owner` adds another number to the array, when calling the `getAttributeProbabilities()` function, it will confuse the `user`. It will also give an advantage to the user when calling the `dnaToIndex()` function, which will allow the user to get the highest `attributeProbabilityIndex` if the sum of all numbers in the nested array is greater than or equal to `rarityRank`

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Proof Of Concept

```solidity
uint8[][] internal _probabilities;
uint8[][] internal _probabilities2;

function getProb() public {
        _probabilities.push([25, 25, 13, 13, 9, 20]);
        _probabilities.push([25, 25, 13, 13, 9, 1]);
        _probabilities.push([25, 25, 13, 13, 9, 10]);
        _probabilities.push([25, 25, 13, 13, 9, 23]);
        _probabilities.push([25, 25, 13, 13, 9, 1]);
        _probabilities.push([25, 25, 13, 13, 9, 3]);
    }

function getProb2() public {
        _probabilities2.push([25, 25, 13, 13, 9, 9, 2]);
        _probabilities2.push([25, 25, 13, 13, 9, 1, 2]);
        _probabilities2.push([25, 25, 13, 13, 9, 10, 2]);
        _probabilities2.push([25, 25, 13, 13, 9, 23, 2]);
        _probabilities2.push([25, 25, 13, 13, 9, 1, 2]);
        _probabilities2.push([25, 25, 13, 13, 9, 1, 2]);
    }

function setUp() public {
        _ownerAddress = address(this);
        _treasuryAddress = vm.addr(1);
        getProb2();

        _fighterFarmContract = new FighterFarm(
            _ownerAddress,
            _DELEGATED_ADDRESS,
            _treasuryAddress
        );

        _helperContract = new AiArenaHelper(_probabilities);
}

function testAddAttributeProbabilitiesFromOwner() public {
        _helperContract.addAttributeProbabilities(0, _probabilities2);
        assertEq(_helperContract.getAttributeProbabilities(0, "head")[6], 2);
    }
```

```bash
forge test --mt testAddAttributeProbabilitiesFromOwner
Running 1 test for test/AiArenaHelper.t.sol:AiArenaHelperTest
[PASS] testAddAttributeProbabilitiesFromOwner() (gas: 211014)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 3.47ms

Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

## Recommendations

I recommend that you add a limit on the length of the nested array, or add a check for the length of the nested array

```diff
+ function addAttributeProbabilities(uint256 generation, uint8[6][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```

or

```diff
function addAttributeProbabilities(uint256 generation, uint8[][] memory probabilities) public {
        require(msg.sender == _ownerAddress);
        require(probabilities.length == 6, "Invalid number of attribute arrays");

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
+          require(probabilities[i].length==6);
            attributeProbabilities[generation][attributes[i]] = probabilities[i];
        }
    }
```

---

## 8. Low-02. Functions accepts an incorrect data type

### Relevant GitHub Links

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L144
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L370

## Summary

Two functions accept an invalid data type as input

## Vulnerability Details

`AiArenaHelper.sol::deleteAttributeProbabilities` and `FighterFarm.sol::reRoll` incorrect input values are specified.Both functions accept `uint8` although they should accept `uint256`

`FighterFarm.sol::reRoll`
`uint8 tokenId`

```solidity
 function reRoll(uint8 tokenId, uint8 fighterType) public { require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");
        ...}
```

This function uses the mapping `numRerolls` which accepts `uint256`

```solidity
mapping(uint256 => uint8) public numRerolls;
```

and

`AiArenaHelper.sol::deleteAttributeProbabilities`

```solidity
 function deleteAttributeProbabilities(uint8 generation) public {
  require(msg.sender == _ownerAddress);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[generation][attributes[i]] = new uint8[](0);
        }
    }
```

This function uses the mapping `attributeProbabilities` which accepts `uint256`

```solidity
mapping(uint256 => mapping(string => uint8[]))
        public attributeProbabilities;
```

## Impact

Incorrect data type will limit the functionality of some functions when input data will be forwarded more than **255**. And it will break the functionality of the whole system.

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

The data type of the function input values should be changed to `uint256`

---

## 9. Low-03. USE OF DEPRECATED SETUPROLE FUNCTION

### Relevant GitHub Links

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L93-L96
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L101-L104
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L109-L112

## Vulnerability Details

"The contract make use of the deprecated function `_setupRole` from the
`AccessControl` contract. As per the [Changelog.md](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ffceb3cd988874369806139ae9e59d2e2a93daec/CHANGELOG.md?plain=1#L351), the function is depre-
cated.

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

It is recommended to use the `_grantRole` function instead

---

## 10. Low-04. No check for address(0)

### Relevant GitHub Links

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L93-L96
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L101-L104
- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L109-L112

## Summary

The function accepting address as input has no check for `address(0)`

## Vulnerability Details

All these functions that take an address as an argument must be checked for address(0)

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

Add a `require(address!=address(0))` check to each function

```diff
function addMinter(address newMinterAddress) external {
        require(msg.sender == _ownerAddress);
+        require(newMinterAddress!=address(0));
        _setupRole(MINTER_ROLE, newMinterAddress);
    }
```

```diff
function addStaker(address newStakerAddress) external {
        require(msg.sender == _ownerAddress);
+       require(newStakerAddress != address(0));
        _setupRole(STAKER_ROLE, newStakerAddress);
    }
```

```diff
function addSpender(address newSpenderAddress) external {
        require(msg.sender == _ownerAddress);
+       require(newSpenderAddress != address(0));
        _setupRole(SPENDER_ROLE, newSpenderAddress);
    }
```

---

## 11. Low-05. No check on the amount of tokens

### Relevant GitHub Links

- https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L127-L134

## Summary

This function does not have a check for the total number of tokens allowed

## Vulnerability Details

If the admin accidentally makes a mistake with the array of provided tokens, or accidentally adds 1 more zero to some value, he can allow a certain user to spend more than needed and more than there is in the `treasuryAddress`.

## Proof Of Concept

```solidity

function testAirDropOverflow() public {
        address[] memory recipients = new address[](2);
        recipients[0] = vm.addr(3);
        recipients[1] = vm.addr(4);
        uint256[] memory amounts = new uint256[](2);
        amounts[0] = 1_000_000_00 * 10 ** 18;
        amounts[1] = 1_500_000_00 * 10 ** 18;
        _neuronContract.setupAirdrop(recipients, amounts);
        uint256 firstRecipient = _neuronContract.allowance(
            _treasuryAddress,
            recipients[0]
        );
        uint256 secondRecipient = _neuronContract.allowance(
            _treasuryAddress,
            recipients[1]
        );
        assertEq(firstRecipient, amounts[0]);
        assertEq(secondRecipient, amounts[1]);
        assertEq(
            _neuronContract.balanceOf(_treasuryAddress),
            2_000_000_00 * 10 ** 18
        );
    }
```

```bash
forge test --mt testAirDropOverflow
Running 1 test for test/Neuron.t.sol:NeuronTest
[PASS] testAirDropOverflow() (gas: 72716)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 1.43ms

Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

## Tools Used

[Forge](https://book.getfoundry.sh/reference/forge/forge)

## Recommendations

You should add a check on the whole array to make them equal to the amount that is in the `treasuryAddress`

```diff
function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
        require(isAdmin[msg.sender]);
        require(recipients.length == amounts.length);
        uint256 totalAmount = 0;
+       for(uint32 i=0;i<amounts.length;i++){
+       totalAmount+=amounts[i];
+        }
+       require(totalAmount<= balanceOf(treasuryAddress));
        uint256 recipientsLength = recipients.length;
        for (uint32 i = 0; i < recipientsLength; i++) {
            _approve(treasuryAddress, recipients[i], amounts[i]);
        }
    }
```

---
