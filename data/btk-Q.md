## Ai Arena Overview

AI Arena is a PvP platform where human-trained AI fighters are tokenized via FighterFarm.sol. Fighters have attributes like appearance, generation, weight, element, type, and model data. Players earn $NRN by entering fighters in ranked battles, facilitated by RankedBattle.sol. Staking tokens and winning battles yields rewards, with potential stake loss for poor performance. Voltage in wallets replenishes to 100 every 24 hours; battles consume 10 voltage units each. Batteries from GameItems.sol restore voltage to 100 or players wait for natural replenishment.

## Findings Summary

| Risk   | Title                                                            |
| ------ | -----------------------------------------------------------------|
| [L-01] | Missing setter function for rerollCost                           |
| [L-02] | Missing setter function for treasuryAddress                      |
| [L-03] | Insecure Elliptic Curve Recovery Mechanism                       |
| [L-04] | Use safeTransferFrom instead of transferFrom for ERC721 trasfers |
| [L-05] | NFTs will have empty URI                                         |
| [L-06] | claimRewards may result in OOG error                             |
| [L-07] | Inability to remove roles in the Neuron contract                 |
| [L-08] | globalStakedAmount will not reflect the  overall staked amount   |

## [L-01] Missing setter function for rerollCost  

## Impact

The absence of a setter function for rerollCost may lead to various issues when the price fluctuates.

## Description

The current implementation hardcodes the rerollCost, which is the price paid by fighters to reroll a new fighter with random traits:

```solidity
uint256 public rerollCost = 1000 * 10**18;    
```

Hardcoding rerollCost can introduce challenges in the future, potentially limiting protocol functionalities. For instance, if the price of the NRN token increases, rerolling may become prohibitively expensive for fighters. Conversely, if the price decreases, rerolling may become too affordable, resulting in an influx of users changing their attributes and DNA simultaneously, potentially congesting the system.

## Recommended Mitigation Steps

Consider adding a setter function to dynamically adjust the rerollCost in response to price changes.

## [L-02] Missing setter function for treasuryAddress   

## Impact

Hardcoding the treasuryAddress in the constructor may lead to a loss of funds for the protocol.

## Description

In the current implementation, treasuryAddress is initialized in the constructor. This address is pivotal as it dictates where funds will be transferred when users reRoll a fighter:

```solidity
bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
```

However, the absence of a setter function for treasuryAddress poses a significant risk. If, at any point in the future, this address is compromised, lost, or controlled by a malicious entity, there is no mechanism in place for the owner or protocol to rectify the situation. As a result, funds will perpetually be directed to the hardcoded address, potentially resulting in irreversible losses.

## Recommended Mitigation Steps

Consider adding a simple setter function to change the treasuryAddress to make the system more robust.

## [L-03] Insecure Elliptic Curve Recovery Mechanism

## Impact

The elliptic curve recovery mechanism utilized in the FighterFarm contract is insecure, posing risks of Malleable Signatures and the potential return of a Null Address.

> [!NOTE]  
> Acknowledgement: The Verification library is OOS. This vulnerability pertains to the in-scope (FighterFarm) contract employing a function from the out-of-scope (Verification) contract.

## Description

FighterFarm.claimFighters function uses Verification.verify to check the validity of the fighter signature, Verification.verify is as follow:

```solidity
    function verify(
        bytes32 msgHash, 
        bytes memory signature,
        address signer
    ) 
        public 
        pure 
        returns(bool) 
    {
        require(signature.length == 65, "invalid signature length");

        bytes32 r;
        bytes32 s;
        uint8 v;

        assembly {
            /*
            First 32 bytes stores the length of the signature

            add(sig, 32) = pointer of sig + 32
            effectively, skips first 32 bytes of signature

            mload(p) loads next 32 bytes starting at the memory address p into memory
            */

            // first 32 bytes, after the length prefix
            r := mload(add(signature, 32))
            // second 32 bytes
            s := mload(add(signature, 64))
            // final byte (first byte of the next 32 bytes)
            v := byte(0, mload(add(signature, 96)))
        }
        bytes memory prefix = "\x19Ethereum Signed Message:\n32";
        bytes32 prefixedHash = keccak256(abi.encodePacked(prefix, msgHash));
        return ecrecover(prefixedHash, v, r, s) == signer;
    }
```

The ecrecover() function is a low-level cryptographic function that should be utilized after appropriate sanitizations have been enforced on its arguments, namely on the s and v values. This is due to the inherent trait of the curve to be symmetrical on the x-axis and thus permitting signatures to be replayed with the same x-value (r) but a different y-value (s).

The ecrecover() function in Solidity has a few potential vulnerabilities that can make it insecure if not handled properly:

- Malleable Signatures: One of the main vulnerabilities is that ecrecover is susceptible to malleable signatures. This means that a valid signature can be transformed into a different valid signature without needing access to the private key. This introduces challenges when relying on signatures for uniqueness or identification.

- Return of Null Address: Another vulnerability is that ecrecover can return a null address (address(0)). This can happen when the recovery ID (v) is set to any value other than 27 or 28. An unsecure contract may allow an attacker to spoof an authorized-only method into executing as though the authorized account is the signer.

## Recommended Mitigation Steps

Consider adopting OpenZeppelin's battle-tested [ECDSA](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol) which incorporates safeguards against both Malleable Signatures and the Return of Null Address vulnerabilities.

## [L-04] Use safeTransferFrom instead of transferFrom for ERC721 transfers

## Impact

The sent NFTs will be locked up in the contract forever.

## Description

Use of transferFrom method for ERC721 transfer is discouraged and recommended to use safeTransferFrom whenever possible by OpenZeppelin. This is because transferFrom() cannot check whether the receiving address know how to handle ERC721 tokens.

For example, If ERC721 token is sent to msg.sender with the transferFrom method and this msg.sender is a contract and is not aware of incoming ERC721 tokens, the sent token could be locked up in the contract forever.

> Reference: https://docs.openzeppelin.com/contracts/3.x/api/token/erc721

## Recommended Mitigation Steps

Use _safeTransfer in transferFrom instead of _transfer:

```solidity
    function transferFrom(
        address from, 
        address to, 
        uint256 tokenId
    ) 
        public 
        override(ERC721, IERC721)
    {
        require(_ableToTransfer(tokenId, to));
        _transfer(from, to, tokenId);
    }
```

## [L-05] NFTs will have empty URI

## Impact

NFT tokens lacking a tokenURI will disrupt off-chain tools reliant on this information, potentially leading to broken functionality or malformed data display. Furthermore, NFT marketplaces utilize the URI to showcase the associated NFT, rendering NFTs with empty URIs unsellable in such platforms.

## Description

The token URI is typically assigned manually by the _delegatedAddress:

```solidity
    function setTokenURI(uint256 tokenId, string calldata newTokenURI) external {
        require(msg.sender == _delegatedAddress);
        _tokenURIs[tokenId] = newTokenURI;
    }
```

However, a function called reRoll is utilized by fighters to generate a new fighter with random traits. reRoll function sets the tokenURI to an empty string:

```solidity
_tokenURIs[tokenId] = ""; 
```

As a result, all URIs will be invalid.

## Recommended Mitigation Steps

Consider initializing the URI when users mint a new fighter and reset it when they reRoll.

## [L-06] claimRewards may result in OOG error

## Impact

The MergingPool.claimRewards function could generate an "Out Of Gas" error.

## Description

In the MergingPool contract, the claimRewards function enables users to batch claim rewards for multiple rounds. It iterates through each round and then through the winnerAddresses array:

```solidity
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
                    claimIndex += 1;
                }
            }
        }
```

When a user is listed in the winnerAddresses, an NFT is minted to them using the mintFromMergingPool function. Internally, _createNewFighter is called, which involves multiple storage reads and invocation of createPhysicalAttributes. 

This function iterates through attribute arrays to generate rarities and attributes, further invoking dnaToIndex, which loops through attribute probabilities. 

While these iterations might not pose an issue with a single winner or round, OOG errors are likely with multiple winners and unclaimed rounds.

## Recommended Mitigation Steps

Consider limiting users to claiming rewards for only one round at a time to mitigate the risk of OOG errors.

## [L-07] Inability to remove roles in the Neuron contract

## Impact

The absence of a role revocation mechanism in the Neuron contract poses a significant risk of access control issues and potential loss of funds.

## Description

The Neuron contract utilizes AccessControl to enforce role-based access control. It employs the internal _setupRole() function to grant roles such as addMinter, addStaker, and addSpender. However, it lacks a corresponding function to revoke these roles. 

```solidity
    function _setupRole(bytes32 role, address account) internal virtual {
        _grantRole(role, account);
    }
```

Consequently, if an address becomes compromised, lost, or controlled by a malicious entity in the future, the contract owner will be unable to revoke the role. This limitation arises from the fact that AccessControl.revokeRole can only be invoked by the role admin. 

```solidity
    function revokeRole(bytes32 role, address account) public virtual override onlyRole(getRoleAdmin(role)) {
        _revokeRole(role, account);
    }
```

Notably, Neuron fails to initialize a role admin for the roles or establish a DEFAULT_ADMIN_ROLE, severely restricting the contract's functionalities. 

Moreover, in the event of a compromised account holding the minter role, the attacker could potentially mint the entire supply without intervention from the contract owner.

## Recommended Mitigation Steps

Consider implementing a role admin for each role using _setRoleAdmin or properly initialize the DEFAULT_ADMIN_ROLE:

```solidity
    function _setRoleAdmin(bytes32 role, bytes32 adminRole) internal virtual {
        bytes32 previousAdminRole = getRoleAdmin(role);
        _roles[role].adminRole = adminRole;
        emit RoleAdminChanged(role, previousAdminRole, adminRole);
    }
```

## [L-08] globalStakedAmount will not reflect the  overall staked amount

## Impact

The globalStakedAmount variable fails to accurately reflect the total staked amount within the RnakedBattle contract.

## Description

During battles, when a fighter loses, the game server executes updateBattleRecord to reward the winner and penalize the loser. In case of a loss, a portion of the loser's staked NRN is slashed and transferred to the StakeAtRisk contract. If the user wins, the stake is reclaimed; otherwise, it is all transferred to treasuryAddress when the round end:

```solidity
    function _sweepLostStake() private returns(bool) {
        return _neuronInstance.transfer(treasuryAddress, totalStakeAtRisk[roundId]);
    }
```

Consequently, even after setNewRound is called and totalStakeAtRisk is transferred to the treasury, globalStakedAmount remains unaltered, rendering it greater than the actual staked amount by fighters.

## Recommended Mitigation Steps

Consider deducting the totalStakeAtRisk from globalStakedAmount when the setNewRound is called.

