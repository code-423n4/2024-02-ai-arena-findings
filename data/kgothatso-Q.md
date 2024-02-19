##Gas optimization
make arrays into constant state variables 

##links 
https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L17

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L20

##Impact
save gas

```diff
-     string[] public attributes = ["head", "eyes", "mouth", "body", "hands", 
     "feet"];
+     string[] public constant attributes = ["head", "eyes", "mouth", "body", 
      "hands", "feet"];


-     uint8[] public defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];
+     uint8[] public  constant defaultAttributeDivisor = [2, 3, 5, 7, 11, 13];

   
```