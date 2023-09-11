## [QA-01] Use ERC1155P instead of ERC1155 for better gas.

ERC1155 is implemented throughout the codebase. ERC1155P is a new implementation of ERC1155 that aims to reduce gas costs for people that are minting, swapping through a burn process, and buying multiple tokens.
Using ERC1155P can significantly reduce the gas costs. 
Since implementing ERC1155P throughout the whole codebase and running gas benchmarks is not feasible in terms of contest audit, this is given as a QA report.

The gas benchmarks and comparison of ERC1155P can be seen from this twitter [thread](https://twitter.com/0xjustadev/status/1620571591653605377) by `justadev`.

The repo for is ERC1155P  [here](https://github.com/0xth0mas/ERC1155P).

## [QA-02] Most of the functions can be frontrun by malicious MEV bots.

There is a chance for functions to be frontrun by malicious MEV bots due to which most access control mechanisms are implemented by another contracts. Like for example in [Delegate.sol::withdraw()](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L353C4-L386C6), 

```solidity
 function withdraw(uint256 delegateTokenId) external nonReentrant {
        bytes32 registryHash = StorageHelpers.readRegistryHash(delegateTokenInfo, delegateTokenId);
        StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, bytes32(StorageHelpers.ID_USED));
        // Sets registry pointer to used flag
        StorageHelpers.revertNotMinted(registryHash, delegateTokenId);
        (address delegateTokenHolder, address underlyingContract) = RegistryHelpers.loadTokenHolderAndContract(delegateRegistry, registryHash);
        StorageHelpers.revertInvalidWithdrawalConditions(delegateTokenInfo, accountOperator, delegateTokenId, delegateTokenHolder);
        StorageHelpers.decrementBalance(balances, delegateTokenHolder);
        delete delegateTokenInfo[delegateTokenId][StorageHelpers.PACKED_INFO_POSITION]; // Deletes both expiry and approved
        emit Transfer(delegateTokenHolder, address(0), delegateTokenId);
        IDelegateRegistry.DelegationType delegationType = RegistryHashes.decodeType(registryHash);
        bytes32 underlyingRights = RegistryHelpers.loadRights(delegateRegistry, registryHash);



/////////////////////////////////////////////
/////////Rest of Function/////////////////////
/////////////////////////////////////////////
```

Most of the access controls are set by another external contract like [StorageHelpers](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol). The MEV bots out there are really intelligent and are able to read the bytecode of a transaction and deploy their own version of `StorageHelpers` if they want to. There is an actual demonstration of this by `Patrick Collins` [here](https://twitter.com/PatrickAlphaC/status/1697038016697499995).

 The `withdraw()` function call by a MEV bot will most likely revert due to not having approval to burn by there are multiple instances of this across the codebase.

So the desired approach is to atleast have one check in the function itself(in this case withdraw), that checks without having any external call.

