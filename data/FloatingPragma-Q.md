# Summary of Findings

| ID       | Title                                                                                         |
|----------|-----------------------------------------------------------------------------------------------|
| NC-1     | Duplicate allowance checks.                                                                   |
| NC-2     | Recipients of airdropped NRN can receive their claim by directly calling transferFrom.       |
| NC-3     | founderAddress could not be the address of the deployer of AAMintPass.                       |
| NC-4     | ecrecover return of 0 address not checked.                                                    |
| LOW-1    | Cumulative allowances may exceed treasury initial mint during NRN airdrop claim.             |

# NC-1

## Lines Of Code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L140

## Duplicate allowance checks.
The check `allowance(treasuryAddress, msg.sender) >= amount,` in the `claim()` method from `Neuron.sol` is explicitly stated before the call to `transferFrom` while in `transferFrom` there is a sub call to the method `spendAllowance` which does the exact same check. The `spendAllowance` method will perform the check that the allowance should be >= `amount` only if we have a concrete allowance (and not an allowance of type(uint256).max). In our case this is sufficient as the allowances are concrete amounts of `NRN` token which represent claims. The check at [L140](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L140) can be removed.

# NC-2

## Lines Of Code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L132
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AAMintPass.sol#L13

## Recipients of airdropped NRN can receive their claim by directly calling transferFrom
The `claim` method in `Neuron.sol` uses `ERC20.transferFrom` method to transfer `NRN` from the treasury to the recipient of the claimed amount. However a user who has claim over the `INITIAL_TREASURY_MINT` can call directly `transferFrom` because he already has a set allowance and in this way the event `TokensClaimed` won't be emitted, potentially messing up off chain tracking. Consider giving the allowances in the `claim` method.

# NC-3

## Lines Of Code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AAMintPass.sol#L13
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AAMintPass.sol#L48

## founderAddress could not be the address of the deployer of AAMintPass
The comments above the state variable `founderAddress` are the following:
`// The founder address is the address that deploys the smart contract`

However in the constructor of `AAMintPass` the value that the `founderAddress` state variable will hold is assigned to the input param `_founderAddress` and not to `msg.sender` meaning an arbitrary address can become the founder if the caller of the constructor does not pass his address as an input. As a result `transferOwnership` can become callable only by the arbitrary address set by mistake, for which it would be unknown who is in control of. Change the code to be `founderAddress = msg.sender;`

# NC-4

## Lines Of Code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Verification.sol#L40

## ecrecover return of 0 address not checked
In `FighterFarm` when the `claimFighters` method is called, a call to `verify` is made to verify the signature of the claim message. However in the logic of `verify` the case where `ecrecover` returns address(0) is not checked. As a result any invalid signature would be treated as valid when paired with a zero address. Perform checks on the return value from `ecrecover()` and revert if the value is 0.


# [LOW-1] Cumulative allowances may exceed treasury initial mint during NRN airdrop claim 

## Lines of code
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L127

## Impact
The `setUpAirdrop` function from `Neuron.sol` is missing a check that would ensure the cumulative allowance is not larger than the tokens available at the treasury address. The function is giving allowances to `recipients[i]` and later when one of these recipients calls `claim` to get his allocated allowance from the treasury, the following scenario can happen:

1. Let's say that the cumulative allowances > the available `NRN` at the treasury.
2. Alice claims her `allowance`, Bob also claims his `allowance` and now the balance of the treasury address is 0, due to miss handling the initial mint.
3. Eve also has an `allowance` to claim but there are not enough tokens in the treasury

Consider keeping track of the cumulative allowance in a memory variable (lets say `cumulativeAllow`) by incrementing it with amounts[i] after each iteration and check that `cumulativeAllow` <= `INITIAL_TREASURY_MINT`.