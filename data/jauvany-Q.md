# 1: PRAGMA VERSION ^0.8.21 TOO RECENT TO BE TRUSTED.

Vulnerability details

## Context:

Unexpected bugs can be reported in recent versions;
Risks related to recent releases
Risks of complex code generation changes
Risks of new language features
Risks of known bugs

0.8.21 (2023-07-19)
0.8.20 (2023-05-10)
0.8.19 (2023-02-22)
0.8.18 (2023-02-01)

For reference, see https://github.com/ethereum/solidity/blob/develop/Changelog.md  

## Proof of Concept

 > ***File: DelegateRegistry.sol***

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L2


 > ***File: RegistryHashes.sol***

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L2


 > ***File: RegistryStorage.sol***

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L2


 > ***File: RegistryOps.sol***

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryOps.sol#L2


> ***File: DelegateToken.sol***

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L2

> ***File: PrincipalToken.sol***

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/PrincipalToken.sol#L2

> ***File: CreateOfferer.sol***

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/CreateOfferer.sol#L2

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Use a non-legacy and more battle-tested version

