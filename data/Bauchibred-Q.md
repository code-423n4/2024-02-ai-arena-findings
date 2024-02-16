# QA Report

## Table of Contents

| Issue ID                                                                                                  | Description                                                                               |
| --------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| [QA-01](#qa-01-a-user-cant-burn-their-tokens-as-they-see-fit)                                             | A user can't burn their tokens as they see fit                                            |
| [QA-02](#qa-02-an-in-game-user-should-in-all-instances-have-access-to-the-amount-of-their-current-points) | An in-game user should in all instances have access to the amount of their current points |
| [QA-03](#qa-03-insecure-spender-approval-mechanism)                                                       | Insecure spender approval mechanism                                                       |
| [QA-04](#qa-04-introduce-protection-against-json-attacks-on-users)                                        | Introduce protection against JSON attacks on users                                        |
| [QA-05](#qa-05-somewhat-flawed-ownership-transfer-in-voltagemanager-sol)                                  | Somewhat flawed ownership transfer in VoltageManager.sol                                  |
| [QA-06](#qa-06-somewhat-flawed-ownership-transfer-in-voltagemanager-sol)                                  | `Mergingpool.sol` currently does not use the VRF which limits the randomization           |

## QA-01 A user can't burn their tokens as they see fit

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-02-ai-arena/blob/befe84a9ddccba0cd0251082046e57ecabffb0cf/src/GameItems.sol#L243-L246

```solidity
    function burn(address account, uint256 tokenId, uint256 amount) public {
        require(allowedBurningAddresses[msg.sender]);
        _burn(account, tokenId, amount);
    }
```

Fundry test case, add this test case to https://github.com/code-423n4/2024-02-ai-arena/blob/befe84a9ddccba0cd0251082046e57ecabffb0cf/test/GameItems.t.sol

```solidity
    function testBurnFromEligibleOwnerDoesNotWork() public {
        _fundUserWith4kNeuronByTreasury(_ownerAddress);
        _gameItemsContract.mint(0, 1);
        assertEq(_gameItemsContract.balanceOf(_ownerAddress, 0), 1);
        vm.expectRevert();
        _gameItemsContract.burn(_ownerAddress, 0, 1);
    }
```

Evdently even as `_ownerAddress` is the owner of this tokenId they can't burn their tokens

### Impact

Flawed implementaton of the burning mechanism, limiting users to what they can do with their tokens.

### Recommended Mitigation Steps

While burning also check if msg.sender is the owner of tokenId and allow them to burn their tokens if they want to.

## QA-02 An in-game user should in all instances have access to the amount of their current points

### Proof of Concept

Take a look at [RankedBattle.sol#L417-L504](https://github.com/code-423n4/2024-02-ai-arena/blob/befe84a9ddccba0cd0251082046e57ecabffb0cf/src/RankedBattle.sol#L417-L504)

The key issue arises below

```solidity
            if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
                /// If the fighter has a positive point balance for this round, deduct points
                points = stakingFactor[tokenId] * eloFactor;
               if (points > accumulatedPointsPerFighter[tokenId][roundId]) {
                    points = accumulatedPointsPerFighter[tokenId][roundId];
                }
                accumulatedPointsPerFighter[tokenId][roundId] -= points;
                accumulatedPointsPerAddress[fighterOwner][roundId] -= points;
                totalAccumulatedPoints[roundId] -= points;
                //@audit
                if (points > 0) {
                    emit PointsChanged(tokenId, points, false);
                }
            } else {
                /// If the fighter does not have any points for this round, NRNs become at risk of being lost
                bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
                if (success) {
                    _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
                    amountStaked[tokenId] -= curStakeAtRisk;
                }
            }
        }
    }
```

Evidently, the event is only emitted in the case whereby a user's point is still above 0, but there are legit instances where a user's point value would be 0 and they would still need to be in the know of the change as seen based on the conditional statement, i.e ` if (points > accumulatedPointsPerFighter[tokenId][roundId]) {points = accumulatedPointsPerFighter[tokenId][roundId];}` this would result to having the points being 0.

### Impact

Incorrect implementation of events for tracking the points in the case where a userr loses their batttle, essentially making it harder for off-chain record keeping

### Recommended Mitigation Steps

Wheever a user loses their battttle and the points get reduced to 0, still emit an event.

## QA-03 Insecure spender approval mechanism

### Proof of Concept

```solidity
function approveSpender(address account, uint256 amount) public {
    require(
        hasRole(SPENDER_ROLE, msg.sender),
        "ERC20: must have spender role to approve spending"
    );
    _approve(account, msg.sender, amount);
}
```

This function allows an address with the `SPENDER_ROLE` to approve a specific amount of tokens to be spent from another address (`account`). However, the critical flaw lies in the fact that the `msg.sender` (who must have the `SPENDER_ROLE`) is the one being approved to spend `amount` from the `account`. This logic inversion means any holder of the `SPENDER_ROLE` can arbitrarily increase their allowance from any account, leading to potential unauthorized token transfers or burns.

#### Exploit Scenario

A malicious actor who has been granted the `SPENDER_ROLE` could call `approveSpender`, passing in the address of a victim (`account`) and the amount they wish to "steal" or burn from the victim's balance. Subsequently, they could use the standard ERC20 `transferFrom` method to transfer the approved amount of tokens from the victim's account to any other account, including their own, effectively draining the victim's balance without consent.

### Impact

The current implementation of the `approveSpender` function in the Neuron contract presents vulnerability that could allow unauthorized token burning and manipulation. By allowing any account with the `SPENDER_ROLE` to set an approval amount for any other account, the system opens up a pathway for potential abuse where a malicious actor with the `SPENDER_ROLE` can arbitrarily approve themselves to spend tokens from other accounts without the token owner's consent.

### Recommended Mitigation Steps

A few ideas

1. **Revise the Approval Logic**: Change the logic of the `approveSpender` function to ensure that only the token owner can initiate an approval for a spender. This aligns with the standard ERC20 approval mechanism where `msg.sender` is the token owner approving a certain allowance for a spender.

2. **Require Direct Owner Approval**: Modify the function to require that `msg.sender` is the account owner wishing to set an allowance for a spender with the `SPENDER_ROLE`. This can be enforced by replacing `account` with `msg.sender` in the `_approve` call and passing the spender's address as a parameter.

3. **Use Secure Roles for Sensitive Actions**: Ensure that roles enabling critical actions like token burning or spending are tightly controlled and assigned only to trusted contracts or entities. Implement additional checks or multi-signature requirements for role assignment.

## QA-04 Introduce protection against JSON attacks on users

### Impact

No special character filtering was applied to the inputted metadata, resulting in `tokenURI` possibly returning data that cannot be correctly parsed into JSON formats and leaving the `tokenUri` potentially vulnerable to a JSON injection attack.

### Proof of Concept

Protocol uses the tokenURI implementation in multiple places, this can be confirmed by using this search command: [https://github.com/search?q=repo%3Acode-423n4%2F2024-02-ai-arena%20tokenURI&type=code](https://github.com/search?q=repo%3Acode-423n4%2F2024-02-ai-arena%20tokenURI&type=code)

Issue with this is that looking through protocol's codes one would realise that there ar eno protoectiuos whatsoever against JSON injection attacks, i.e for the metadata any character, including special characters can be passed into the details

### Recommended Mitigation Steps

Cosider checking for special characters to prevent something of this nature from happening

## QA-05 Somewhat flawed ownership transfer in VoltageManager.sol

### Proof of Concept

Take a look at the `transferOwnership` function within the VoltageManager contract and other similar implementations. Although this function correctly updates the `_ownerAddress` to a new owner, it fails to update the `isAdmin` mapping to reflect this change in ownership. Consequently, the previous owner retains their admin status even after the ownership has been officially transferred.

```solidity
function transferOwnership(address newOwnerAddress) external {
    require(msg.sender == _ownerAddress, "Only the owner can transfer ownership");
    _ownerAddress = newOwnerAddress;
    // Missing logic to revoke admin access from the previous owner
}
```

Additionally, the contract does not have a mechanism to automatically revoke the admin status of the previous owner or to grant admin status to the new owner, leading to potential security risks.

### Impact

Borderline low, cause this is somewhat a malicious admin issue and the new admin can still change the isAdmin status of the old one, but that one might as well just front run this attempt with running any malicious operation they can withthe role.

Now the primary security concern is the retained admin access by the previous owner, which could lead to unauthorized adjustments of critical contract parameters, including but not limited to:

- Manipulating the list of allowed voltage spenders.
- Adjusting other admins' access inappropriately.
- Potential misuse of other admin-only functionalities not explicitly outlined in the provided code snippet.

Such actions could undermine the contract's operational integrity, potentially leading to loss of user trust, manipulation of gameplay mechanics, or unauthorized access to in-game assets.

### Recommended Mitigation Steps

Modify the `transferOwnership` function to include the logic to revoke the previous owner's admin access. This can be achieved by setting `isAdmin[_ownerAddress] = false;` before updating the `_ownerAddress` to the new owner.

## QA-06 `Mergingpool.sol` currently does not use the VRF which limits the randomization

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-02-ai-arena/blob/befe84a9ddccba0cd0251082046e57ecabffb0cf/src/MergingPool.sol

We can see that the code's natspec and [protocol's provided docs](https://docs.aiarena.io/gaming-competition/merging), do not always align, for example in the docs, it's stated that the merging pool is to use the chainlink VRF for random numbers while picking the winner, but this is no where implemed in code.

### Impact

Low, cause after a discussionn with sponsors it's been considered an "out-of-date" documentation case, otherwise one could perceive this as medium cause protocol does not pick winner as intended since the randomization has been reduced and causes picking the winner to be more predictable.

### Recommended Mitigation Steps

Make sure that the docs are always up to date
