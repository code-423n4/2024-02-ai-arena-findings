1、https://github.com/code-423n4/2024-02-ai-arena/blob/main/script/Deployer.s.sol#L66-L69
2、https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L147
3、https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L250

In the deployment script Deployer.s.sol, after creating the fighterFarm contract, the instantiateMintpassContract function is not called to initialize _mintpassInstance, causing the call to the redeemMintPass function to fail.