### [G-1] 'FighterOps.sol::Fighter::generetion' type uint16 is gas cheaper
 Code snipped:
 https://github.com/code-423n4/2024-02-ai-arena/blob/70b73ce5acaf10bc331cf388d35e4edff88d4541/src/FighterOps.sol#L36-L46

 When run test in 'FighterFarm.t.sol': 

      function testGetAllFighterInfo() public {
        _mintFromMergingPool(_ownerAddress);
        uint256 gasUsed = gasleft();
        (, , , , , , uint16 generation) = _fighterFarmContract
            .getAllFighterInfo(0);
        console.log(
            "Gas used for getting AI generation: ",
            gasUsed - gasleft()
        );
    }

 with 'FighterOps.sol::Fighter::generetion' type uint8 we got this output:

[PASS] testGetAllFighterInfo() (gas: 439097)
Logs:
  Gas used for getting AI generation:  12113


But when type is changed to uint16 we got output:

[PASS] testGetAllFighterInfo() (gas: 439089)
Logs:
  Gas used for getting AI generation:  12099

Which means that changing 'FighterOps.sol::Fighter::generetion' from uint8 to uint16 safes 14 gas to the users who would like to execute 'FighterFarm.sol::getAllFighterInfo'

    struct Fighter {
        uint256 weight;
        uint256 element;
        FighterPhysicalAttributes physicalAttributes;
        uint256 id;
        string modelHash;
        string modelType;
        - uint16 generation;
        + uint8 generation;
        uint8 iconsType;
        bool dendroidBool;
    }
