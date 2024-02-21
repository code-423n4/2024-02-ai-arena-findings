## [G-1] Use Merkle Tree

Use Merkle Tree for airdropping tokens. This will save a decent amount of gas and will require only storing the root inside the contract.

```solidity
    function setupAirdrop(address[] calldata recipients, uint256[] calldata amounts) external {
        require(isAdmin[msg.sender]);
        require(recipients.length == amounts.length);
        uint256 recipientsLength = recipients.length;
        for (uint32 i = 0; i < recipientsLength; i++) {
            _approve(treasuryAddress, recipients[i], amounts[i]);
        }
    }

    /// @notice Claims the specified amount of tokens from the treasury address to the caller's address.
    /// @param amount The amount to be claimed
    function claim(uint256 amount) external {
        require(allowance(treasuryAddress, msg.sender) >= amount, "ERC20: claim amount exceeds allowance");
        transferFrom(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }

```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L127-L145

## [G-2] Sum at the end

Updating global variables inside a loop is quite expensive. Here it is possible to add the value once.

```diff
    function claimNRN() external {
        require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
        uint256 claimableNRN = 0;
        uint256 nrnDistribution;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution)
                / totalAccumulatedPoints[currentRound];
            // @gas we can sum up at the end
-           numRoundsClaimed[msg.sender] += 1;
        }
+       numRoundsClaimed[msg.sender] += roundId - lowerBound;
        if (claimableNRN > 0) {
            amountClaimed[msg.sender] += claimableNRN;
            // @high time bomb probably, we should transfer from the treasury
            _neuronInstance.mint(msg.sender, claimableNRN);
            emit Claimed(msg.sender, claimableNRN);
        }
    }
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L294-L311

## [G-3] Address saved twice

The contract stores the address and instance, which is redundant. Simply use `StakeAtRisk(_stakeAtRiskAddress)` where needed. The `_stakeAtRiskInstance` can be removed.

```solidity
contract RankedBattle {
	...
	address _stakeAtRiskAddress;
	...
	StakeAtRisk _stakeAtRiskInstance;
}
```
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L69