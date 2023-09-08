# Lines of code

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L2C1-L2C24
# Vulnerability details

## Impact

The contract specifies the pragma solidity^0.8.4; on the compiler before version 0.8.18, such arguments cannot be used in functions, as indicated: the function ```
```solidity
function writeApproved(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, address approved) internal { ... }
```

`mapping(uint256 delegateTokenId => uint256[3] info)`
similar mapping in arguments is present in 15 functions. 
## Proof of Concept

compile v0.8.17 and below log:
```
Retrieving compiler information:
Compiler using remote version: 'v0.8.17+commit.8df45f5f', solidity version: 0.8.17+commit.8df45f5f.Emscripten.clang
ParserError: Expected '=>' but got identifier
  --> d:/1SOLIDITY/2023-09-delegate/src/libraries/DelegateTokenStorageHelpers.sol:37:44:
   |
37 |  ... tion writeApproved(mapping(uint256 delegateTokenId => uint256[3] info) storage delega ...
   |                                         ^^^^^^^^^^^^^^^

Compilation failed with 1 errors
```
compile v0.8.18 log:
```
Retrieving compiler information:
Compiler using remote version: 'v0.8.18+commit.87f61d96', solidity version: 0.8.18+commit.87f61d96.Emscripten.clang
Compilation completed successfully!
```
## Tools Used

VS Code

## Recommended Mitigation Steps

change from `pragma solidity ^0.8.4;` to `pragma solidity ^0.8.18;`