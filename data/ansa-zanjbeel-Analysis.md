## 🔗 **OVERVIEW**

- In `AI ARENA` You Can Train An `AI Character` To Battle In a Platform Fighting Game.
- The Characters Are AIs, & You Can Train Them To Learn Any Skill In Preparation For Battle.
- AI Arena Is A `PVP` Platform, Where The Fighters Are AIs That Were Trained By Humans.
- In The Web3 Version Of This Game, These Fighters Are Tokenized Via The `FighterFarm.sol (smart contract)`

## (Each `Fighter NFT` Contains the Following)

```

- Physical Attributes: Determines The Visual Appearance.
- Generation: This Primarily Affects The Visual Appearance.
- Weight: Determines The Battle Attributes.
- Element: Determines Its Special Abilities.
- Fighter Type: Indicates Whether It's a Regular Champion/Dendroid.
- Model Data: Comprising Of The Model Type & Model Hash.

```

- Players Can Enter their NFT Fighters Into A Ranked Battle To Earn Rewards In Native Token $NRN.
- Token Is An ERC20 Token, As Defined In The `Neuron.sol (smart contract)`
- During deployment, `RankedBattle.sol (smart contract)` is granted The `MINTER and STAKER` Roles To Facilitate The Reward System.
- `FighterFarm.sol & GameItems.sol` Are Granted The `SPENDER Role` To Allow For In-Game Purchases.
- Players Are Only Able To Earn $NRN, By Staking Their Tokens & Winnings.
- However, `Important To Note` That It Is Possible For Players To Lose Part Of Their Stake If They Perform Poorly.
- To Level The Playing Field, We Take The Square Root Of The Amount Staked To Calculate The `StakingFactor`, Which Is Used In The Points Calculation After Each Ranked Match.
- Each Wallet Has `Voltage` That It Has To Manage.
- `Every 24 Hours` From The Start Of Their First Initiated Match For The Day, Voltage Will Be Replenished `Back To 100`
- If A Player Runs Out Of Voltage They Either Have To Wait Until It Naturally Replenishes Or They Can Purchase a Battery From The `GameItems.sol`
- Each Battery Will Fill Voltage `Back to 100`
- Each Ranked ` Battle Costs 10 Voltage Units`

## 🔗 **IN-GAME REWARD SYSTEM**

**(TWO TYPES OF REWARDS)**

```

There Are Two Types Of Rewards:

1. NRN
2. NFT

```

**(REWARD DEPENDS ON TWO THINGS)**

```

The Amount You Earn Depends On Two Things:

1. Performance In Arena
2. How Many NRN's You Have Staked

```

## **(FOUR CONCEPTS IN REWARD SYSTEM)**

**CONCEPT # 1**

![alt text](<IMAGE 1.jpg>)

- At The Core Of The Reward System There Is A Function Called `NRN STAKING`
- `Staking NRN Carries Risk`, Which Helps To Enforce Proper Incentives For Competitors In The Rank Battles.
- To Start, You Must Stake NRNs Behind Your NFTs.
- If You Don't Stake Any NRN Your NFT Won't Earn Reward.
- The Higher You Stake, The More Potential Reward.
- However, There's Also A `Higher Risk Of Losing NRNs If You Perform Poorly`

**CONCEPT #2**

![alt text](<../IMAGE 2.jpg>)


- Now The System Is Actually Slightly More Involved Than What Was Shown Aove When You Compete In Rank Battles, You Do Not Earn NRNs Tokens Directly.- There's An `Intermediate System Called Points`
- After Each Round Of Competition, A Fixed Pool Of NRNs Tokens Is Distributed As Rewards.
- This Is Called The `NRNs Reward Pool`
- Your Share Of This Pools Depends On How Many Points Your NFT Can Earn During That Round.
- Think Of Points As a Claim On The Reward Pool.
- The More Points You Have Relative To Competitors, The Bigger Your Claim Is On The Pool.
- For Example, If The Reward Pool Has 1000 NRNs As Rewards and Your NFTs Earn 10% Of All Points In That Round Of Competition, You Will Receive 100 NRN As Rewards.

  `Now Points Are Calculated On Three Drivers:`

- One, Whether Your NFT Wins or Losses Fights
- Two, how much NRN You've Staked On Your NFT
- Three, Your NFT's Elo Ranking

All Else Being Equal, More Wins Means More Points, & The Higher The NRN Stake and The Higher Your Elo Rating Mean Greater Points.

**CONCEPT # 3**

![alt text](<../IMAGE 4.jpg>)

- So Far We've Covered Scenarious When You Win But `What If You Lose?`
- Losing Fights Can Mean `Different Things Depending On Where You Are`
- Sometimes It Means Losing Points, And Other Times, You're Potentially `Going To Lose NRN`
- When Your Point balance is positive, you're in what's called the points earning zone.
- When Its Negative , You Are Now At Risk Zone.

`Now, Let's Look At A Few Examples`

```

- When Your Point Balance Is Poitive And You Win, You Are Earning Points.
- When It's Positive And You Lose, You Are Losing Points.
- However, This All Changes When Your Point Balance Reaches Zero.
- If You Continue To Lose, Your NRNs Start To Become At Risk.
- The More You Lose, The More NRNs Is At Risk.
- If You End A Round Of Competition With NRNS At Risk, You Will Lose It Permanently! :(
- But Wait There Is Still A Chance!
- Even If You Do Have NRNs At Risk And You Start Winning Again, You Can Recover Them Back Before The Round Ends.

```

**CONCEPT # 4**

- There's One More Thing You Can Do With Points.
- You Can Allocate Some To The `Merging Pool` For A Chance To Win A New NFT.
- In The Staking Module Of The Game, You Can Pre Select A Percentage Of Your Points To Allocate To The Merging Pool.
- Your Chances Of Winning Is Dependent On `How Many Points` You Have Staked Relative To The Total Points In The Pool.
- This Is Basically Similar To How The `NRN Reward Pool` Work As Well.
- Points Are Only Deducted From Your Balance If You `Win An NFT`
- If You Don't Win, Your Points Carry Over To Future Cycles.

## **AI ARENA ANALYSIS**

| List | Contract               | SLOC | Details                                                                                                                                                                                                                           |
| :--- | :--------------------- | :--- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | src/AiArenaHelper.sol  | 95   | This contract generates and manages an AI Arena fighters physical attributes                                                                                                                                                      |
| 2    | src/FighterFarm.sol    | 327  | This contract manages the creation, ownership, and redemption of AI Arena Fighter NFTs, including the ability to mint new NFTs from a merging pool or through the redemption of mint passes.                                      |
| 3    | src/FighterOps.sol     | 74   | This library defines the Fighter struct and contains methods for fetching information about a fighter.                                                                                                                            |
| 4    | src/GameItems.sol      | 163  | This contract represents a collection of game items used in AI Arena.                                                                                                                                                             |
| 5    | src/MergingPool.sol    | 110  | This contract allows users to potentially earn a new fighter NFT.                                                                                                                                                                 |
| 6    | src/Neuron.sol         | 92   | The Neuron token is used for various functions within the platform, including staking, governance, and rewards.                                                                                                                   |
| 7    | src/RankedBattle.sol   | 300  | This contract provides functionality for staking NRN tokens on fighters, tracking battle records, calculating and distributing rewards based on battle outcomes and staked amounts, and allowing claiming of accumulated rewards. |
| 8    | src/StakeAtRisk.sol    | 63   | This contract allows the RankedBattle contract to manage the staking of NRN tokens at risk during battles.                                                                                                                        |
| 9    | src/VoltageManager.sol | 47   | This contract allows the management of voltage for game items and provides functions for using and replenishing voltage.                                                                                                          |

## ANALYSIS OF THE CODE BASE

## a) Admin Abuse Risks

There Are Some Administrative Risks. Since The Contract Allows Ownership Transfer, Improper Handling Could Increase The Possibility Of Inappropriate Transfer Of Ownership, Which Could Result In Misuse Or Manipulation Of The Contract's Functions. Also The Owner Is In Charge Of The Fighter Generations Incrementation, Which Could Be Abused If Improperly Monitored.

## b) Systemic Risks

The Owner's Privileges are Mainly Used By Contracts To Handle Different Functions. This May Introduce Systemic Risks, Including Single Points Of Failure and Susceptibility To Malicious Activities If The Owner's Address Is Compromised. The Contract Relies On Various External Calls. Any Vulnerabilities In These Contracts May Impact The Functionality.

## c) Technical Risks

There Are Some Technical Risks Such As Arrays Without Explicit Checks On Their Sizes, Loops May Lead To Higher Gas Costs And Potential Scalability Issues If Not Handle Properly. Also External Function Calls Should Be Monitered Carefully.

## d) Integration Risks

As The Contract Interacts With Other External Contracts So, Integration Risks May Arise Due To Compatibility Issues, Incorrect Data, Or Differient Contract Standards.


## **Hope This Help! Best Of Luck!**



### Time spent:
18 hours