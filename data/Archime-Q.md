1、 In the deployment script Deployer.s.sol, after creating the fighterFarm contract, the instantiateMintpassContract function is not called to initialize _mintpassInstance

https://github.com/code-423n4/2024-02-ai-arena/blob/main/script/Deployer.s.sol#L66-L69     
     fighterFarm.setMergingPoolAddress(address(mergingPool));
     fighterFarm.instantiateAIArenaHelperContract(address(aiArenaHelper));
     fighterFarm.instantiateNeuronContract(address(neuron));
     fighterFarm.addStaker(address(rankedBattle));


2、The instantiateAIArenaHelperContract function is not called and _aiArenaHelperInstance is not instantiated.
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L147
    function instantiateAIArenaHelperContract(address aiArenaHelperAddress) external {
        require(msg.sender == _ownerAddress);
        _aiArenaHelperInstance = AiArenaHelper(aiArenaHelperAddress);
    }

3、In the function redeemMintPass, the functions ownerOf, burn, etc. in the _mintpassInstance instance will be called. Because _mintpassInstance is not assigned a value, the execution of the redeemMintPass function will fail.
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L250
function redeemMintPass(
        uint256[] calldata mintpassIdsToBurn,
        uint8[] calldata fighterTypes,
        uint8[] calldata iconsTypes,
        string[] calldata mintPassDnas,
        string[] calldata modelHashes,
        string[] calldata modelTypes
    ) 
        external 
    {
        ......
        for (uint16 i = 0; i < mintpassIdsToBurn.length; i++) {
            require(msg.sender == _mintpassInstance.ownerOf(mintpassIdsToBurn[i]));
            _mintpassInstance.burn(mintpassIdsToBurn[i]);
        ......
        }
    }
