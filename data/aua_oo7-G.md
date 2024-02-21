### [G-1]`require()` strings longer than 32 bytes cost extra gas

### Proposed Optimization - used custom error instead.

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/AiArenaHelper.sol#L133

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/MergingPool.sol#L196

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/Neuron.sol#L139

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/Neuron.sol#L156

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/Neuron.sol#L157

### [G-2]Add unchecked {} for subtractions where the operands can't underflow because of previous checks.

Issue Description-The subtraction operation uint256 decreasedAllowance = allowance(account, msg.sender) - amount; does not have an unchecked block. even though underflow is not possible due to the previous check  require(allowance(account, msg.sender) >= amount,"ERC20: burn amount exceeds allowance"); on line 197-200. adding an unchecked block helps avoid unnecessary check.

Proposed Optimization:- Add an unchecked block around the subtraction:

unchecked{

uint256 decreasedAllowance = allowance(account, msg.sender) - amount;

}

Estimated Gas Saving:- Adding an unchecked block for a subtraction that cannot underflow due to a previous check saves approximately 3 gas per subtraction. with many such subtractions occurring across interactions the total gas savings could be significant.

https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/Neuron.sol#L201

### [G-3] Caching the length of calldata  increases gas cost instead of reducing it.

issue 1- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L122

function pickWinner(uint256[] calldata winners) external {

require(isAdmin[msg.sender]);

require(winners.length == winnersPerPeriod, "Incorrect number of winners");

require(!isSelectionComplete[roundId], "Winners are already selected");

uint256 winnersLength = winners.length;

address[] memory currentWinnerAddresses = new address[](winnersLength);

for (uint256 i = 0; i < winnersLength; i++) {

currentWinnerAddresses[i] = _fighterFarmInstance.ownerOf(winners[i]);

totalPoints -= fighterPoints[winners[i]];

fighterPoints[winners[i]] = 0;

}

winnerAddresses[roundId] = currentWinnerAddresses;

isSelectionComplete[roundId] = true;

roundId += 1;}

### Proposed Optimizations- use calldata variable length directly without saving in local variable to save gas.

--uint256 winnersLength = winners.length;

--address[] memory currentWinnerAddresses = new address[](winnersLength);

--for (uint256 i = 0; i < winnersLength; i++) {

+ +address[] memory currentWinnerAddresses = new address[](winners.length);

+ + for (uint256 i = 0; i < winners.length; i++) {

issue 2- https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L130

function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {

require(isAdmin[msg.sender]);

require(recipients.length == amounts.length);

uint256 recipientsLength = recipients.length;

for (uint32 i = 0; i < recipientsLength; i++) {

_approve(treasuryAddress, recipients[i], amounts[i]);

}

}

### Proposed Optimization- use the calldata length directly without saving it in local variable.

- - uint256 recipientsLength = recipients.length;

-- for (uint32 i = 0; i < recipientsLength; i++) {

+ + for (uint32 i = 0; i < recipients.kength; i++) {

_approve(treasuryAddress, recipients[i], amounts[i]);

}

### [G-04] State variables should be cached in stack variables rather than re-reading them from storage

**Issue Description** - In the `stakeNRN()` function, the state mapping variable `amountStaked` is read directly from storage on line 253,256. This is inefficient, as it incurs an extra storage read operation that is unnecessary.

**Proposed Optimization** - Cache the current value of amountStaked in a local stack variable before using it and updating storage. This avoids an unnecessary re-read of the storage.

```
function  changeUnwrapFee(uint256  nextUnwrapFeeDivisor) external  override  onlyOwner {
 uint256  currentUnwrapFeeDivisor = unwrapFeeDivisor;
 if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
 emit  ChangeUnwrapFee(currentUnwrapFeeDivisor, nextUnwrapFeeDivisor, msg.sender);
 unwrapFeeDivisor = nextUnwrapFeeDivisor;
}
```

**Estimated Gas Savings** - Reading a storage slot costs roughly 200 gas. Caching the value in a local variable avoids this extra read, saving roughly 200 gas per function call.

**Code Snippet:**

```
File: src/ocean/Ocean.sol
196  function  changeUnwrapFee(uint256  nextUnwrapFeeDivisor) external  override  onlyOwner {
197  /// @notice as the divisor gets smaller, the fee charged gets larger
198  if (MIN_UNWRAP_FEE_DIVISOR > nextUnwrapFeeDivisor) revert();
199  emit  ChangeUnwrapFee(unwrapFeeDivisor, nextUnwrapFeeDivisor, msg.sender); //@audit >>> unwrapFeeDivisor is state-variable
200  unwrapFeeDivisor = nextUnwrapFeeDivisor;
201 }
1082 (transferAmount, truncated) = _convertDecimals(NORMALIZED_DECIMALS, decimals, amount); //@audit >>>
1097  _convertDecimals(decimals, NORMALIZED_DECIMALS, transferAmount); //@audit >>>
```