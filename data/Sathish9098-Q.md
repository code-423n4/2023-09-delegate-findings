# LOW FINDINGS

##

## [L-1] Hardcoded ``interfaceId`` values may cause problem in future 

Hardcoding the ``interfaceId`` in the ``supportsInterface`` function may cause problems in the future. This is because the ``interfaceId`` is an ``arbitrary number`` that can be changed at ``any time``. If the ``interfaceId ``changes, the ``supportsInterface`` function will no longer work correctly.

```solidity
FILE: 2023-09-delegate/src/DelegateToken.sol

function supportsInterface(bytes4 interfaceId) external pure returns (bool) {
        return interfaceId == 0x2a55205a // ERC165 Interface ID for ERC2981
            || interfaceId == 0x01ffc9a7 // ERC165 Interface ID for ERC165
            || interfaceId == 0x80ac58cd // ERC165 Interface ID for ERC721
            || interfaceId == 0x5b5e139f // ERC165 Interface ID for ERC721Metadata
            || interfaceId == 0x4e2312e0; // ERC165 Interface ID for ERC1155 Token receiver
    }

```

```solidity
FILE: delegate-registry/src/DelegateRegistry.sol

330: return Ops.or(interfaceId == type(IDelegateRegistry).interfaceId, interfaceId == 0x01ffc9a7);

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L330

### Recommended Mitigation
You can use a dynamic approach to get the interfaceId. One way to do this is to use the IERC165 interface. The IERC165 interface defines a function called supportsInterface that returns true if the contract supports the specified interface

```solidity

function supportsInterface(bytes4 interfaceId) external pure returns (bool) {
  // Get the interfaceId of the contract that we are calling.
  bytes4 contractInterfaceId = IERC165(msg.sender).supportsInterface(interfaceId);

  // Return true if the contract supports the specified interface.
  return contractInterfaceId == interfaceId;
}

```
## [L-2] Ensure that the setApprovalForAll function checks for the address(0) to prevent the possibility of the operator being set to address(0)

The setApprovalForAll function does not check if the operator is address(0). This means that it is possible for the owner of the contract to approve address(0) to transfer all of their tokens on their behalf .However, there are also some risks associated with approving address(0) to transfer tokens on your behalf. For example, if you approve address(0) to transfer all of your tokens, then anyone could steal your tokens

```solidity
FILE: 2023-09-delegate/src/DelegateToken.sol

function setApprovalForAll(address operator, bool approved) external {
        accountOperator[msg.sender][operator] = approved;
        emit ApprovalForAll(msg.sender, operator, approved);
    }

```
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L144-L147

### Recommended Mitigation
Add address(0) check

```
require(operator != address(0));

```
##
## [L-3] Array lengths not checked

If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array.

```solidity
FILE: 2023-09-delegate/src/CreateOfferer.sol

function previewOrder(address caller, address, SpentItem[] calldata minimumReceived, SpentItem[] calldata maximumSpent, bytes calldata context)
        external
        view
        onlySeaport(caller)
        returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)
    {
        if (context.length != 160) revert Errors.InvalidContextLength();
        (offer, consideration) = Helpers.processSpentItems(minimumReceived, maximumSpent);
    }

```
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/CreateOfferer.sol#L176-L184

### Recommended Mitigation
Add length check

```solidity
require(minimumReceived.length == maximumSpent.length, " Length not matched" ).

````

##
## [L-4] Signature use at deadlines should be allowed

According to EIP-2612, signatures used on exactly the deadline timestamp are supposed to be allowed. While the signature may or may not be used for the exact EIP-2612 use case (transfer approvals), for consistency's sake, all deadlines should follow this semantic. If the timestamp is an expiration rather than a deadline, consider whether it makes more sense to include the expiration timestamp as a valid timestamp, as is done for deadlines.

```solidity
FILE: Breadcrumbs2023-09-delegate/src/DelegateToken.sol

341: if (StorageHelpers.readExpiry(delegateTokenInfo, delegateTokenId) < block.timestamp) {

```
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L341

```solidity
FILE: 2023-09-delegate/src/libraries/DelegateTokenStorageHelpers.sol

162: if (block.timestamp < readExpiry(delegateTokenInfo, delegateTokenId)) {

```
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L162

##

## [L-5] Latest solidity versions 0.8.21 is not compatible with all other chains 

The latest Solidity version, 0.8.21, is not compatible with all other chains. Some chains, such as Ethereum Classic and Binance Smart Chain, are still using older versions of Solidity.

```solidity
FILE: delegate-registry/src/DelegateRegistry.sol

2: pragma solidity ^0.8.21;

FILE: Breadcrumbsdelegate-registry/src/libraries/RegistryHashes.sol

2: pragma solidity ^0.8.21;

```
##

## [L-6] Return values of ``transferFrom()`` not checked

Not all IERC20 implementations revert() when there's a failure in transfer()/transferFrom(). The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually making a payment

```solidity
FILE: File: src/DelegateToken.sol

369: IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);

393: IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);

```
https://github.com/code-423n4/2023-09-delegate/blob/87dda32e96e5249e51bcd1b5dd53361d7c794694/src/DelegateToken.sol#L369

```solidity
FILE: File: src/libraries/DelegateTokenTransferHelpers.sol

41: IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);

```
https://github.com/code-423n4/2023-09-delegate/blob/87dda32e96e5249e51bcd1b5dd53361d7c794694/src/libraries/DelegateTokenTransferHelpers.sol#L41

##

## [L-7] Avoid using vulnerable version ``openzeppelin-4.9.0``

Known Vulnerabilities

- Improper Encoding or Escaping of Output
- Improper Input Validation
- Missing Authorization

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/fd81a96f01cc42ef1c9a5399364968d0e07e9e90/contracts/token/ERC721/ERC721.sol#L2

### Recommended Mitigation
Use ``openzeppelin-contracts version 4.9.3`` consistently for all contracts 

##

## [L-8] ``safeMint()`` should be used rather than ``mint()`` wherever possible 

_mint() is [discouraged] in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function. In the cases below, ``_mint()`` does not call ``ERC721TokenReceiver.onERC721Received()`` on the recipient.

```solidity
FILE: 2023-09-delegate/src/libraries/DelegateTokenStorageHelpers.sol

75:  PrincipalToken(principalToken).burn(msg.sender, delegateTokenId);

```
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L75























































