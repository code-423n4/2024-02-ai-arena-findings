# Analysis report

## general notes

- transferOwnership, adjustAdminAccess and other functions are used multiple times, they could just as well be put in a library.
- use modifiers instead of adding the same require statements everywhere, preferably use access control from openzeppelin, this will drastically improve readability as well as gas usage.
- use revert message in require statements so it's easier to debug
- Add reentrancy guard and stick to the checks-effects-interactions pattern to protect against reentrancy attacks, I haven't found anything that could be exploited but you will be more at ease knowing an attack like this is not possible.
  

### Time spent:
11 hours