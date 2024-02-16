- There are soo many places in the code where access control is needed, the team kept writing require statements repetitively. Repetition is prone to error which might be hard to debug.. it is advisable to use a modifier as best practise and to better improve the readibility of the code.


- There were numerous require statements in the codebase where there's no string argument specifying why the revert happens. I personally spent almost 24hrs debugging a revert test just because team doesnt add a string statement for the require function. It is advisable to add reasons why a function is reverting.

from solidity doc:
```
If you do not provide a string argument to require, it will revert with empty error data, not even including the error selector.
```