1) `setTokenURI` in `FighterFarm` contract is vulnerable to XSS
If a malicious admin sets the tokenURI to `javascript:alert()` or a `data` based XSS payload, it may lead to cross-site scripting in front-end, where the link is imported in web2 websites
