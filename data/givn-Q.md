### [L-01] Improve security posture of `_delegatedAddress` in `FighterFarm.sol`

The `_delegatedAddress` field represents the backend's address, responsible for signing various messages related to `claimFighters` .  There are two issues with it, which increase the risk for the protocol.

1. `_delegatedAddress` is being set only once, in the constructor, and there is no check if its `address(0)`. In the off-chance of this value being misconfigured any signature would be able to mint a fighter.
2. In the event of a security breach on the backend, there is no way to change the `_delegatedAddress`. This means a hacker could mint as many and whatever kind of fighter they would like.

**Suggested mitigation**
- Add require check in `FighterFarm`'s constructor
```diff
+ require(_delegatedAddress != address(0), "Invalid delegated address");
```
- Add  `setDelegatedAddress(address) external` function, which should be callable only by the owner, similar to the already existing `setMergingPoolAddress(address)`.

### [L-02] Missing functions to change the treasury in various contracts

The `treasuryAddress` receives an initial mint of NRN and subsequently smaller transfers of the currency. 

In the situation of a security breach, protocol ownership change or some other unforeseen events it could be troublesome that the treasury address can't change. A fallback mechanism is necessary, just like one exists for changing the **owner**.

**Suggested mitigation**
Either add `setTreasury(address)` function, callable only by the owner, in the following contracts:
- `FighterFarm.sol`
- `GameItems.sol`
- `Neuron.sol`
- `StakeAtRisk.sol` 

Or make all of them reference the treasury from the `Neuron` contract, since they all have a reference to it, and have `setTreasury(address)` only in `Neuron.sol`.
### [L-03] Malicious players can use winning machine learning model, `FighterFarm.sol`

Due to the fact that machine learning hashes and types are public, as part of `FighterOps.Fighter[] public fighters;`, and `updateModel` can be called with arbitrary model & type data, the following scenario is possible:

> Malicious player finds a fighters with a winning strategies and uses them to progress in the current round.

This allows an attacker to piggy back on someone else's strategy and save time in researching themselves.

**Suggested mitigation**
Option 1: Since model hashes are unique, the protocol could keep track if someone is trying to set an already set hash.
Option 2: Sign the message on the backend to ensure users can't set arbitrary ai models.

### [L-04] Missing checks for game item invariants when calling `createGameItem`

It is possible for an item to have infinite supply (`finiteSupply == false`) and have `itemsRemaining > 0`.

Add:
```sol
if(finiteSupply && itemsRemaining > 0) {
  revert("Conflicting item parameters");
}
```

### [L-05] There are no functions for adjusting `GameItemAttributes` 

In the event of misconfigured item, promotions/discounts, balance adjustments a new item with the same name needs to be created. Adding function to update parameter/s is going to make resolution of such situations straight-forward.

### [L-06] `getFighterPoints` view function is unusable 

The `getFighterPoints` function is created as a helper, to produce a list of the fighters' points. The issue with it is that it allocates the resulting array with only 1 item. This makes it impossible to use for any practical purposes, because client code would be interested in more fighters than the first one.

Another concern with the function is that it could become impossible to use if enough fighters are created. A big enough array would DoS the function. Consider providing it with a range of indices - $0..<FightersCount$. In this way you would limit the return data.

#### PoC
```sol
function testIndexOOB() public {
  address user1 = makeAddr("user1");
  address user2 = makeAddr("user2");
  _mintFromMergingPool(_ownerAddress);
  _mintFromMergingPool(user1);
  _mintFromMergingPool(user2);

  vm.startPrank(address(_rankedBattleContract));
  _mergingPoolContract.addPoints(0, 100);
  _mergingPoolContract.addPoints(1, 100);
  _mergingPoolContract.addPoints(2, 100);
  assertEq(_mergingPoolContract.totalPoints(), 300);
  vm.stopPrank();
  
  // fighter points contains values for 3 fighters, but when we ask for 2 or 3, it fails
  vm.expectRevert(stdError.indexOOBError);
  uint256[] memory fighterPoints = _mergingPoolContract.getFighterPoints(2);
  assertEq(fighterPoints.length, 2);
}
```


### [NC-01] Misleading function names in `FighterFarm.sol`

The following functions should be renamed:
`instantiateAIArenaHelperContract` to `setAIArenaHelperContract`
`instantiateMintpassContract` to `setMintpassContract`
`instantiateNeuronContract` to `setNeuronContract`

Since they are not actually instantiating anything, `set` would be a more appropriate prefix. Just like it is done in `setMergingPoolAddress` and `setTokenURI`.

### [NC-02] Extract constant for custom attributes in `FighterFarm.sol`

As mentioned by the developers the value 100 is used for custom attributes:
> guppy — 02/11/2024 3:50 PM
   100 is a flag we use to determine if it’s a custom attribute or not

To avoid sprinkling literals in the codebase and improve it's readability it is recommended to extract this literal to a constant, something like this:
```sol
/// @notice Special value for custom attributes
uint256 private constant CUSTOM_ATTRIBUTE = 100
```

And replacing it in the following lines: [here](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L219), [here](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L259) and [here](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L499)

### [NC-03] Rename `dendroidBool` to `isDendroid`

Variable containing its type is redundant in statically typed languages. `is` prefix is common convention for boolean values.

Alternatively, replace the value with `uint8 fighterType`.

### [NC-04] Fix misleading function name and natspec in `GameItems.sol`

The function `instantiateNeuronContract` should be renamed to `setNeuronContract`, because it conveys what the function actually does.

Additionally, update the comment, to be more accurate:
```diff
- /// @notice Sets the Neuron contract address and instantiates the contract.
+ /// @notice Sets the Neuron contract address.
```

### [NC-05] Send zero bytes as data argument when minting 

It's unusual to have an arbitrary string as the data parameter of the `_mint` function. Replace `bytes("random")` with `"0x"`

### [NC-06] Missing notice tag in natspec for variables in `MergingPool.sol`

```diff
- /// The address that has owner privileges (initially the contract deployer).
+ /// @notice The address that has owner privileges (initially the contract deployer).
address _ownerAddress;

- /// The address of the ranked battle contract.
+ /// @notice The address of the ranked battle contract.
address _rankedBattleAddress;
```

> Default tag is @notice, but it's nice to be consistent with the other natspecs.
### [NC-06] Incorrect doc string for variable in `MergingPool.sol`

```diff
- /// @notice Mapping of address to fighter points.
+ /// @notice Mapping of fighterId to fighter points.
mapping(uint256 => uint256) public fighterPoints;
```

### [NC-07] Common setup code in tests can be extracted to a base class

Currently, there is a lot of duplicated setup code and variables in the tests. See the setup of:
- `AiArenaHelper.t.sol`
- `FighterFarm.t.sol`
- `MergingPool.t.sol`
- `RankedBattle.t.sol`
- `StakeAtRisk.t.sol`
- `VoltageManager.t.sol`

This can be avoided by creating a Base contract that contains common setup and then each Test contract will do only its own specific configuration.

### [NC-08] Misleading function names in `RankedBattle.sol`

Rename `instantiateNeuronContract` to `setNeuronContract`, to reflect what the function is actually doing.
Update the natspec:
```diff
- /// @notice Instantiates the neuron contract.
+ /// @notice Sets the neuron contract.
```

Rename `instantiateMergingPoolContract` to `setMergingPoolContract`, to reflect what the function is actually doing.
Update the natspec:
```diff
- /// @notice Instantiates the merging pool contract.
+ /// @notice Sets the merging pool contract.
```

Rename `setNewRound()` to `beginNewRound()`. As prefix set implies the caller will specify a roundId, which is not the case. The function actually increments the roundId.

### [NC-09] Improvements to `updateBattleRecord` function

Create a `BattleResult` enum and use it instead of `uint8`. This will improve the code readability. It will be easy to understand if we're looking in the case of a `win` and additional comments like `/// If the user won the match` will become unnecessary, as the code will be self-documenting.

```diff
+ enum BattleResult {
+   win,
+   tie,
+   loss
+ }

function updateBattleRecord(
  ...
- uint8 battleResult,
+ BattleResult battleResult,
  ...
)
```

Rename `updateBattleRecord` parameter:
```diff
function updateBattleRecord(
  ...
- bool initiatorBool,
+ bool isInitiator,
  ...
)
```
It will convey better the purpose of the variable.

### [NC-10] Simplify assert statements

Some assert statements are being unnecessarily complicated, there are overloads that support all cases for comparison of native types. Using the correct APIs increases the readability and expressiveness of code. There are many instances where assert calls can be improved, below I have outlined some general examples.

In the following lines, the `==` operator can be omitted:
```diff
- assertEq(_mergingPoolContract.winnerAddresses(0, 0) == _ownerAddress, true);
+ assertEq(_mergingPoolContract.winnerAddresses(0, 0), _ownerAddress);

- assertEq(mintPass.totalSupply() == mintPass.numTokensOutstanding(), true);
+ assertEq(mintPass.totalSupply(), mintPass.numTokensOutstanding());

```

Here a specialised assert can be used:
```diff
- assertEq(mintPass.mintingPaused(), true);
+ assertTrue(mintPass.mintingPaused());
```

> See `forge-std/lib/ds-test/src/test.sol` for all assert overloads.

Avoid using the Solidity native assert statement, when you can use the APIs from Forge:
```diff
- assert(_helperContract.attributeToDnaDivisor("head") != 99);
+ assertNotEq(_helperContract.attributeToDnaDivisor("head"), 99);
```

### [NC-11] Consider replacing decimal multiplier with `ether` keyword

Since ether contains 18 decimals it is possible to use the `ether` keyword when describing NRN values. It is shorter and conveys the amount just as well.

Example:
```diff
- uint256 stakeAmount = 3_000 * 10 ** 18;
+ uint256 stakeAmount = 3_000 ether;
```

A simple string replace will update all instances correctly, with the exception of 1, which is easy to do manually.