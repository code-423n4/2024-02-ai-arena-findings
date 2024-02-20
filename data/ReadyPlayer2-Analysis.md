QA Report

ISSUE(ADMIN RISK) #1

The unmitigated risk of admin error or abuse. The function to transfer ownership in the Fighter Farm contract and the Arena Helper amongst other contracts in the protocol lacks a two step process for the transference of admin rights and a zero check to prevent accidental or malicious renouncing of admin rights.

Solution

Set up a two step process to transfer admin rights, first by setting a pending admin, then requiring that pending admin to claim ownership. Finally add a check against address zero to prevent renouncing of admin rights by error. If renouncing admin rights is a required function, then adding a function explicitly to renounce ownership shall suffice.

ISSUE(CENTRALIZATION) #2

The level of trust placed on the admins of the protocol creates a single point of failure, that is the protocol is dependent on external input from the admins to keep the protocol running, such as the set new round function in Ranked battle or the pick winner function in the merging pool. Although the assumption is that admins are trusted, it is clear that such will not always be the case, problems such as synchronicity (more than one admin decides to set a new round), consensus (do all off-chain parties agree to this decision) and centralization (how is the admin always guaranteed to deliver) can cause major problems for the protocol in the future.

Solution

I would recommend that the admin role be only used to adjust state variables, that is variables that do not interfere with game logic, game logic updates in the contract should be independent from external factors as much as possible, even though games are run on an off-chain server. For example, to start a new round, a time lapse period can be used. 

Also I would highly recommend a design overhaul in which the off-chain server is treated as an oracle and only the fight results are needed from it. A user can start fights on-chain and the contract can queue a fight request to the server via event emission. Later the server can then call the contract and update all necessary information.

There is no best solution for such a hybrid design, but if there are numerous centralized off-chain entities required for the protocol to function, then this causes numerous points of failure, however if the protocol where to essentially create and ensure a single off-chain point of failure, then they could better pool their resources to manage the protocol efficiently.




### Time spent:
14 hours