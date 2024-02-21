# L-01 :Users will loss batteries even if voltage replenish time is over if `useVoltageBattery` is called ! 

## Impact
Users will loss batteries even if voltage replenish time is over if `useVoltageBattery` is called ! 

## Proof of Concept
```solidity 
   function useVoltageBattery() public {
        require(ownerVoltage[msg.sender] < 100);
        require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
        _gameItemsContractInstance.burn(msg.sender, 0, 1);
        ownerVoltage[msg.sender] = 100;
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }

```
function `useVoltageBattery` is for using battery to replenish Voltage . but it doesnot check if voltage replenish time is over or not . Because of this , even if voltage replenishTime is over , it will cost battery to user . 
From a game logical perspective , this is not a fair scenario . Voltage should be replenished every 24 hour even if user calls  `useVoltageBattery` .

## Tools Used
Manual review ! 
## Recommended Mitigation Steps
Check if voltage replenishtime is over or not : 
Re-write the function as below : 
```solidity 
   function useVoltageBattery() public {
        require(ownerVoltage[msg.sender] < 100);
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }else {
           require(_gameItemsContractInstance.balanceOf(msg.sender, 0) > 0);
           _gameItemsContractInstance.burn(msg.sender, 0, 1);
            ownerVoltage[msg.sender] = 100;
        }  
        emit VoltageRemaining(msg.sender, ownerVoltage[msg.sender]);
    }

```

# L-02 : minter role check should be placed before supply check 
By standard practices , access control chcecks are placed before any other check . But here its not done in this way ! 
```solidity 
   function mint(address to, uint256 amount) public virtual {//@audit-issue minter role check should be placed before supply check 
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
        require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        _mint(to, amount);
    }
```
Code link : https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L155C1-L159C6
### Recommendation :
Re-write the function in this way : 
```diff 
 function mint(address to, uint256 amount) public virtual {
         require(hasRole(MINTER_ROLE, msg.sender), "ERC20: must have minter role to mint");
        require(totalSupply() + amount < MAX_SUPPLY, "Trying to mint more than the max supply");
       
        _mint(to, amount);
    }
```
# L-3 : no event emission in `setNewRound() ` function . 
It is standard to emit an event after important state changes 
```solidity 
 function setNewRound() external {
        require(isAdmin[msg.sender]);
        require(totalAccumulatedPoints[roundId] > 0);
        roundId += 1;
        _stakeAtRiskInstance.setNewRound(roundId);
        rankedNrnDistribution[roundId] = rankedNrnDistribution[roundId - 1];//adopts previous distribution
    }//@audit-issue  no event emission ! 
```
Code link : https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L233C1-L239C6
### Recommendation :

```diff 
 //event declaration 
 event NewRoundStarted(uint256 , uint256)

 function setNewRound() external {
        require(isAdmin[msg.sender]);
        require(totalAccumulatedPoints[roundId] > 0);
        roundId += 1;
        _stakeAtRiskInstance.setNewRound(roundId);
        rankedNrnDistribution[roundId] = rankedNrnDistribution[roundId - 1];
+       emit NewRoundStarted(roundID , rankedNrnDistribution[roundID]);
    }
```
