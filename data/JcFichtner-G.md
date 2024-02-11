## [G-01] Use vanity addresses as function arguments (safely!)

In the **FighterFarm** contract there are several functions that receive as arguments new contract addresses. It is cheaper to use vanity addresses with leading zeros, this saves calldata gas cost.

***A good example is OpenSea Seaport contract with this address:***  
> 0x00000000000000ADc04C56Bf30aC9d3c0aAF14dC.

This will not save gas when calling the address directly. However, if that contract’s address is used as an argument to a function, that function call will cost less gas due to having more zeros in the calldata.

This is because the EVM gas cost for transaction data is determined by the bytes in the transaction. Each non-zero byte in transaction data costs more gas than a zero byte.

When a vanity address, which contains leading zeros, is passed as an argument in a function call, it can lead to a reduction in the overall gas cost of the transaction. This is because the leading zeros in the address take up less gas to encode in the calldata than non-zero bytes would.

This is also true of passing EOAs with a lot of zeros as a function argument – it saves gas for the same reason.

Just be aware that there have been hacks from generating vanity addresses for wallets with insufficiently random private keys. This is not a concert for smart contracts vanity addresses, because smart contracts do not have private keys.

***This would be most relevant for:*** 

> function instantiateAIArenaHelperContract(address aiArenaHelperAddress) external

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L147

> function instantiateMintpassContract(address mintpassAddress) external

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L155

> function instantiateNeuronContract(address neuronAddress) external

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L163

> function setMergingPoolAddress(address mergingPoolAddress) external

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L171

When these addresses are used as function arguments, the overall gas cost could be reduced.

## [G-02] ERC1155 can provide significant optimizations over using separate ERC-721 and ERC-20 contracts

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol

Using ERC-1155 for a game where players mint their NFT fighters ***(FighterFarm.sol)*** to earn rewards in the token $NRN ***(Neuron.sol)*** can indeed provide significant optimizations over using separate ERC-721 and ERC-20 contracts. The ERC-1155 standard is designed for efficiency and flexibility, supporting both fungible and non-fungible tokens within a single contract. This multi-token standard can lead to gas savings and a more streamlined contract architecture for your game. Here are some reasons why ERC-1155 is a better choice for your use case:

1 - ***balanceOf function:*** The ERC721 balanceOf function is rarely used in practice but adds a storage overhead whenever a mint and transfer happens. ERC1155 tracks balance per id, and also uses the same balance to track ownership of the id. If the maximum supply for each id is one, then the token becomes non-fungible per each id.

2 - ***Batch Transfers:*** ERC-1155 allows for batch transfers of multiple token types in a single transaction. This can significantly reduce gas costs when players need to transfer multiple types of assets (e.g., different NFT fighters or tokens representing in-game items and rewards) simultaneously.

3 - ***Reduced Contract Complexity:*** Since ERC-1155 can handle both fungible (like $NRN tokens) and non-fungible tokens (like NFT fighters), you only need to deploy and manage one smart contract instead of two separate contracts (one for ERC-721 NFTs and another for ERC-20 tokens). This simplifies development and interaction with the contract.

4 - ***Gas Efficiency:*** The ERC-1155 standard is designed with gas efficiency in mind. Operations that would typically require multiple transactions and contract interactions in separate ERC-721 and ERC-20 setups can be combined into fewer transactions with ERC-1155, saving gas.

5 - ***Simplified Ownership and Balance Tracking:*** ERC-1155 provides a unified way of tracking ownership and balances for both fungible and non-fungible tokens, making it easier to implement features like ranked battles and rewards distribution within your game.

6 - ***Advanced Functionality:*** ERC-1155 supports more complex features, such as safe batch transfers and approval settings, which can be leveraged to create a more secure and user-friendly gaming experience.

7 - ***Interoperability:*** By adhering to a standard that supports both fungible and non-fungible token types, your game can achieve greater interoperability with wallets, exchanges, and other games or platforms that support ERC-1155.

By adopting ERC-1155 for your game, you can achieve a more gas-efficient, flexible, and streamlined asset management system that benefits both the developers and the players.

## [G-03] Use ERC20-Permit to batch the approval and transfer step in on transaction

If the developers still choose to use ERC-721 and ERC-20 instead of ERC-1155, they can implement ERC-721A (Recommended in the bot report) and ERC20-Permit.

The ERC-20 Permit feature introduces a more gas-efficient way for token holders to grant spending allowances to other addresses. This method leverages digital signatures to allow token holders to approve an allowance without executing a transaction themselves, thus not paying any gas fees for the approval. The recipient of the permit can then use the permit to submit the transaction, including the transfer, in a single step. This mechanism not only saves gas but also improves user experience by streamlining interactions with the contract.

By leveraging the ERC-20 Permit feature, you can make your game's token interactions more efficient and user-friendly, even when choosing to stick with separate ERC-721 and ERC-20 contracts. This approach combines the benefits of ERC-20's flexibility with optimizations that reduce gas costs and simplify token management​​.

***In the link below there is a guide that will walk you through on how to use ERC-20 permit approvals.***

https://www.quicknode.com/guides/ethereum-development/transactions/how-to-use-erc20-permit-approval

  




