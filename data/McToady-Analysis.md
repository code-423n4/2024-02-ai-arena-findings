# AI Arena

## Project Overview
The AI Arena project is a game that allows users to train 'fighters' to compete with other fighters with game rules akin to Super Smash Bros.
However the AI components as well as the mechanics of the game all exist offchain so are not part of this contracts review.

The project uses the blockchain for the following:
- The ingame currency $NEURON (ERC20)
- The collectable fighters that compete in the game (ERC721)
- Ingame items that can be bought/used/sold/traded (ERC1155)
- Keeping a permanent record of a fighters `RankedBattle` record
- Creating/storing the pseudorandom attributes of each 'fighter' ERC721 
- Allowing users to wager Neuron on their Ranked Battle success, giving them the chance to earn/lose Neuron
- Allowing users who've waged Neuron to earn points to have the opportunity to win another collectable fighters.

The overall aim of this analysis is to assess the project's integration of these goals and give guidance on how things could be improved, as well as issues they should be aware of. 

## Architecture Overview Diagram
The diagram below outlines the general relationship between the contracts in scope as well as a brief overview of their key responsibilities.

[Diagram](https://imgur.com/a/QNcAOSq)

## Security Review Approach
When undertaking this security review the following steps were taken:
1. Understand what the protocol sets out to do.
2. Map out where and how users will interact with the protocol.
3. Take note of any areas where the code's result may differ from either the protocol's intentions or the expectations of the user.
4. List out all the scenarios where the code might result in unexpected outcomes.
5. Using a combination of manual review and specific code tests, go through the list of potential issues and attempt to validate/invalidate each.

## General Risks

### The project demands a large amount of trust from users
The project's team have the ability to do all of the following things:
- Mint new unlimited fighters
- Mint unlimied $NEURON (by giving out the `MINTER_ROLE`)
- Spend users $NEURON (by giving out the `SPENDER_ROLE` or `STAKER_ROLE`)
- Start new rounds in `RankedBattle` as frequently/infrequently as they wish.
- Adjust the transferability of `GameItems`
- Change the stored address of one of the projects other contract's (for examle (`instantiateNeuronContract` in GameItems.sol)

As well as this the initial `_owner` set in contracts with an admin role is also given the admin role. Meaning the owner always has complete access to all of a contract's privaleged functions.

**Recommendations**
- If possible try to limit the amount of power the team to make changes over the protocol.
- Ensure contracts with two separate access controlled roles don't have single addresses that hold both powers simultaneously.
- Ensure the addresses with the powers that could cause the most damage are controlled by a multisig smart contract wallet rather than a single EOA.

### Ensuring the onchain and offchain components of the game remain in sync
Within `RankedBattle.sol` the `_gameServerAddress` is responsible for calling `updateBattleRecord` which has an effect on the onchain results tracking for a fighter, the fighters voltage level, and most importantly the staked $NEURON in the system. Should there be any reason a user's battle record does not get updated correctly it could cause them to miss out on Neuron to claim or lose the Neuron they already have at stake.

Potential causes to be wary of:
- The protocol's server responsible for pushing `updateBattleRecord` txs goes down.
- An issue with Arbitrum or the RPC used by the game server means transaction's aren't posted.
- The potential of a chain reorganisation that excludes previously complete transactions.
- `_gameServerAddress` runs out of ETH to pay gas.
- The `updateBattleRecord` tx reverts unexpectedly.

The project needs contingencies in place should any of the above happen to ensure that any transactions that failed to go through are made again later when the problem is resolved.

**Recommendation**
It's important that the game server is robust in ensuring that not only are transactions sent once a battle is completed, but that they also ensure that the transactions are successful. It could be useful to add an event to `updateBattleRecord` so the server responsible for updating the record can be sure the transaction is complete. It would also be recommended to wait a certain amount of blocks for confirmation to avoid any ill effects of a chain reorganisation.

### Limiting actions of a single address
The protocol currently has checks based on the state of a single address, notably `MAX_FIGHTERS_ALLOWED` and `ownerVoltage`. These checks limit a single address rather than a single person, as they can easily transfer fighters to another address to bypass those checks. The protocol team should remain aware of this and ensure that users who engage in this behaviour don't get any unintended benefits from doing so.

## Codebase Comments
A number of steps could be taken to improve the overall quality of the code:
- The codebase is currently lacking a lot of events in functions that change state. Adding events may make it easier for the project's offchain systems to keep in sync with the onchain portions of the project.
- The require statements in the project's code often lack error messages. This can make it very difficult for a user to understand why a transaction is reverting. It also makes it harder for the developer to debug the code in testing.
- There's a large amount of code reused across multiple contracts, particularly in the implementation of as typical `owner` features being present in each of the contracts rather than inheriting them. Also the contract doesn't use `onlyOwner` or `onlyAdmin` modifiers but instead repeats the same require statement multiple times in each contract. Limiting this code reuse would reduce the bloat in the codebase and make the code review experience much smoother.
- The [projects documentation](https://docs.aiarena.io/) appears to be out of date and what is stated there often does not match with the reality of the coded provided for this contest. The team should update their documentation to improve the project's auditability but more importantly as to not deceive potential investors. 

### Time spent:
30 hours