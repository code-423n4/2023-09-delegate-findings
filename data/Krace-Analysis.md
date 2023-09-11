# Auther: Krace


# Approach

- Read the Readme.md to get an initial understanding of the project.
  - This is a V2 version of delegate including delegate registry and delegate marketplace
  - The delegate registry is an immutable database that aggregates onchain programmable access control. It's similar to V1 but expands some more functions.
  - The delegate marketplace wraps delegation rights into ERC721 tokens that can then be traded or transferred. Users could deposit NFT to get DelegationToken.
    - DelegateToken. delegate rights 
    - PrincipalToken. redeem rights
    - CreateOfferer. gasless listing of DelegateTokens
- Clone code and set up the test environment 
  - Include two project: [registry-lib](https://github.com/delegatexyz/delegate-registry/tree/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1) and [delegate](https://github.com/code-423n4/2023-09-delegate/tree/main)
- Read the bot-report
- Analyzing the smart contract's logic 
  - Focus on Common Vulnerability Patterns: reentrancy attacks, integer overflow/underflow, and unauthorized access to sensitive functions or data.
  - Examined the use of external dependencies, ensuring that they were appropriately implemented and that their security implications were understood and accounted for.
  - Reviewed the contract's permissions and access control mechanisms, verifying that only authorized users had access to critical functions and data.
  - Simulated various scenarios, such as edge cases and unexpected inputs, to assess how the contract handled exceptional situations.


# Codebase quality analysis

## [DelegateRegistry.sol](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol)
### Analysis
This is the core library of delegation registry, including three main parts:
- delegate to all 5 kinds of delegaions
- validate the delegation registry
- return the delegaions registry info



## [RegistryHashes.sol](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol)
### Analysis
Use assembly code to calculate 5 types of delegate registry hashes, the hash is shifted left by one byte and the last byte is then encoded with a delegation type.



## [RegistryStorage.sol](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol)
### Analysis
`RegistryStorage` is responsible for packing and unpacking the address of `from`, `to` and `contract`.
There are two slots to store the packed data:
``firstPack := contract[-20:-12] + from[-20:]``
``secondPack := contract + to[-20:]``



## [RegistryOps.sol](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryOps.sol)
### Analysis
Provide normal `max`, `&&` and `||` operations implemented in assembly.



## [DelegateToken.sol](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol)
### Analysis
The implementation of DT token, overrides and extends the ERC721 standard.



## [DelegateTokenLib](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol)
### Analysis
Checks for DT's Flashloan, ERC721ReceiverCallback and Expiry.



## [DelegateTokenRegistryHelpers.sol](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol)
### Analysis
This library primarily implements two functions: loading data and verifying hash.
- Loading data(contract, to, from, amount, tokenId, rights) according to hash
- Verify the provided info with the provided hash. Including flashloan, transfer, delegate info.

### Problem
There are unchecked addition and subtraction operations in the contract. Although the comments assume that overflow or underflow won't occur, considering this is a library contract, it might be better to provide users with some reminders about potential overflows or underflows.

## [DelegateTokenStorageHelpers.sol](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol)
### Analysis
- Read and write to the map `delegateTokenInfo`
- Modify the map `balances`
- Mint and burn `principal` token, check it's approved status.

### Problem
There is a lot of redundant code in the contract. I believe you should reuse this code by using function calls instead of rewriting it.
For example:
```
    function revertInvalidWithdrawalConditions(
        mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo,
        mapping(address account => mapping(address operator => bool enabled)) storage accountOperator,
        uint256 delegateTokenId,
        address delegateTokenHolder
    ) internal view {
        //slither-disable-next-line timestamp
        if (block.timestamp < readExpiry(delegateTokenInfo, delegateTokenId)) {
            if (delegateTokenHolder == msg.sender || msg.sender == readApproved(delegateTokenInfo, delegateTokenId) || accountOperator[delegateTokenHolder][msg.sender]) {
                return;
            }
            revert Errors.WithdrawNotAvailable(delegateTokenId, readExpiry(delegateTokenInfo, delegateTokenId), block.timestamp);
        }
    }
```
The `revertInvalidWithdrawalConditions` could be rewritten to :
```
    function revertInvalidWithdrawalConditions(
        mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo,
        mapping(address account => mapping(address operator => bool enabled)) storage accountOperator,
        uint256 delegateTokenId,
        address delegateTokenHolder
    ) internal view {
        //slither-disable-next-line timestamp
        if (block.timestamp < readExpiry(delegateTokenInfo, delegateTokenId)) {
             revertNotApprovedOrOperator(accountOperator, delegateTokenInfo, delegateTokenHolder, delegateTokenId));
        }
    }
```
But this change will also revert `NotApproved` rather than `WithdrawNotAvailable`, maybe you should write the `IF statement` to a stand alone internal function.


## [DelegateTokenTransferHelpers](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol)
### Analysis
Library that checks the transfer arguments for `ERC20`, `ERC721` and `ERC115`.


## [PrincipalToken](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol)
## Analysis
Mint and burn `PrincipalToken`.

## Problem 
`_checkDelegateTokenCaller` should be changed to a modifier.



## [CreateOffererLib](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol)
### Analysis
The Library provides modifier and helper functions for `CreateOffer`.
### Problem
1. Since this is a Library, if function `processSpentItems` only supports vector with 1 element, I think it should be rewritten to pass `SepntItem` directly rather than the vector.
2. `calculateOrderHash` will `|` the hash and `type`, if users wrongly pass a type which is larger than 8 bit, it will overwrite the hash.



## [CreateOfferer](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol)
### Analysis
There are three stages in Offerer:
- generate. only Seaport can generate order
- transfer. anyone can transfer, the from address must be `this` and every transfer will create a delegateToken.
- ratify. only Seaport can ratify order




# Mechanism review
1. `delegate-registry` library. This library is in charge of calculating and storing the delegate registry hash.
2. DelegateToken. Anyone can deposit `ERC721`, `ERC20` or `ERC115` to mint a `DelegateToken`, and can delegate this token to anyone they want. When minting `DelegateToken`, a corresponding `PrincipalToken` will also be minted, which records the owner of `DelegateToken`.
3. Offer. Seaport could generate orders, and anyone could take the order by calling transferfrom, which will also create a `DelegateToken`.


# Systemic risks
After conducting an audit of this project, the most notable observation is the presence of a lot of duplicated logic in the library. I believe many interfaces could be consolidated to various extents, significantly reducing code redundancy.
Additionally, there may be some shortcomings in the system's flash loan functionality, as mentioned in my other reports. Flash loans might not be able to handle certain unexpected scenarios.


# Time spent
30 hours

### Time spent:
30 hours