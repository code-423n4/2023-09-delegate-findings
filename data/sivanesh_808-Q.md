## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Return values of ``approve`` not checked | 1 |
| [L-2](#L-2) | Array is push()ed but not pop()ed | 2 |
| [L-3](#L-3) |  Solidity version 0.8.20 may not work on other chains due to PUSH0 | 5 |
| [L-4](#L-4) | Governance functions should be controlled by time locks | 4 |
| [L-5](#L-5) | Direct supportsInterface() calls may cause caller to revert | 2 |
| [L-6](#L-6) | Timestamp may be manipulation | 5 |
| [L-7](#L-7) | Some tokens may revert when large transfers are made | 8 |
| [L-8](#L-8) | Some tokens may revert when zero value transfers are made | 8 |
| [L-9](#L-9) | Unbounded state variable arrays exist within the project which are iterated upon | 2 |
| [L-10](#L-10) | Unsafe ERC20 operation(s) | 4 |

### [L-1] Return values of ``approve`` not checked
Not all ``IERC20`` implementations ``revert`` when there a failure in ``approve``. The function signature has a boolean return value and they indicate errors that way instead.By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything.

*Instances (1)*:

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L121

```solidity
File: example/CreateOfferer.sol

121:             if (!IERC20(erc20Order.info.tokenContract).approve(address(delegateToken), erc20Order.amount)) {

```
### [L-2] Array is push()ed but not pop()ed
Array entries are added but are never removed. Consider whether this should be the case, or whether there should be a maximum, or whether old entries should be removed. Cases where there are specific potential problems will be flagged separately under a different issue.

*Instances (2)*:

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L339C1-L340C59

```solidity
File: example/DelegateRegistry.sol

339:         outgoingDelegationHashes[from].push(delegationHash);

340:         incomingDelegationHashes[to].push(delegationHash);

```

### [L-3]  Solidity version 0.8.20 may not work on other chains due to PUSH0
The compiler for Solidity 0.8.20 switches the default target EVM version to Shanghai, which includes the new PUSH0 op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier EVM version. While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

*Instances (5)*:
```

```solidity
File: example/CreateOffererLib.sol

2: pragma solidity ^0.8.4;

```
```solidity
File: example/DelegateTokenLib.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: example/DelegateTokenRegistryHelpers.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: example/DelegateTokenStorageHelpers.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: example/DelegateTokenTransferHelpers.sol

2: pragma solidity ^0.8.4;

```

### [L-4] Governance functions should be controlled by time locks
Governance functions (such as upgrading contracts, setting critical parameters) should be controlled using time locks to introduce a delay between a proposal and its execution. This gives users time to exit before a potentially dangerous or malicious operation is applied.

*Instances (4)*:
```solidity
File: example/CreateOfferer.sol

55:         onlySeaport(msg.sender)

74:         onlySeaport(msg.sender)

179:         onlySeaport(caller)

```

```solidity
File: example/CreateOffererLib.sol

169:     modifier onlySeaport(address caller) {

```

### [L-5] Direct supportsInterface() calls may cause caller to revert
Calling supportsInterface() on a contract that doesnt implement the ERC-165 standard will result in the call reverting. Even if the caller does support the function, the contract may be malicious and consume all of the transactions available gas. Call it via a low-level staticcall(), with a fixed amount of gas, and check the return code, or use OpenZeppelins ERC165Checker.supportsInterface().In the example below, the function call to supportsInterface() may cause _validateMigrationTarget() to revert, even though the contract may otherwise be fully functional, and fallback checks would have allowed the targeted operation.

*Instances (2)*:

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L329

```solidity
File: example/DelegateRegistry.sol

329:     function supportsInterface(bytes4 interfaceId) external pure returns (bool) {

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L65

```solidity
File: example/DelegateToken.sol

65:     function supportsInterface(bytes4 interfaceId) external pure returns (bool) {

```

### [L-6] Timestamp may be manipulation
The block.timestamp can be manipulated by miners to perform MEV profiting or other time-based attacks.

*Instances (5)*:

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L328

```solidity
File: example/CreateOffererLib.sol

328:             return block.timestamp + expiryLength;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L341

```solidity
File: example/DelegateToken.sol

341:         if (StorageHelpers.readExpiry(delegateTokenInfo, delegateTokenId) < block.timestamp) {

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L109

```solidity
File: example/DelegateTokenLib.sol

109:         if (expiry > block.timestamp) return;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L162C4-L168C6

```solidity
File: example/DelegateTokenStorageHelpers.sol

162:         if (block.timestamp < readExpiry(delegateTokenInfo, delegateTokenId)) {

166:             revert Errors.WithdrawNotAvailable(delegateTokenId, readExpiry(delegateTokenInfo, delegateTokenId), block.timestamp);

```

### [L-7] Some tokens may revert when large transfers are made
Tokens such as COMP or UNI will revert when an address balance reaches type(uint96).max. Ensure that the calls below can be broken up into smaller batches if necessary.

*Instances (7)*:
```solidity
File: example/CreateOfferer.sol

89:     function transferFrom(address from, address targetTokenReceiver, uint256 createOrderHashAsTokenId) external checkStage(Enums.Stage.transfer, Enums.Stage.ratify) {

```

```solidity
File: example/DelegateToken.sol

180:             RegistryHelpers.transferERC721(delegateRegistry, registryHash, from, newRegistryHash, to, underlyingRights, underlyingContract, underlyingTokenId);

184:             RegistryHelpers.transferERC20(

198:             RegistryHelpers.transferERC1155(

369:             IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);

393:             IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);

```

```solidity
File: example/DelegateTokenTransferHelpers.sol

41:         IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);

```

### [L-8] Some tokens may revert when zero value transfers are made
Despite the fact that EIP-20 states that zero-value transfers must be accepted, some tokens, such as LEND, will revert if this is attempted, which may cause transactions that involve other tokens (such as batch operations) to fully revert. Consider skipping the transfer if the amount is zero, which will also save gas.REFER:"https://github.com/ethereum/EIPs/blob/7500ac4fc1bbdfaf684e7ef851f798f6b667b2fe/EIPS/eip-20.md?plain=1#L116

*Instances (8)*:
```solidity
File: example/CreateOfferer.sol

54:         checkStage(Enums.Stage.generate, Enums.Stage.transfer)

89:     function transferFrom(address from, address targetTokenReceiver, uint256 createOrderHashAsTokenId) external checkStage(Enums.Stage.transfer, Enums.Stage.ratify) {

```

```solidity
File: example/DelegateToken.sol

180:             RegistryHelpers.transferERC721(delegateRegistry, registryHash, from, newRegistryHash, to, underlyingRights, underlyingContract, underlyingTokenId);

184:             RegistryHelpers.transferERC20(

198:             RegistryHelpers.transferERC1155(

369:             IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);

393:             IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);

```

```solidity
File: example/DelegateTokenTransferHelpers.sol

41:         IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);

```

### [L-9] Unbounded state variable arrays exist within the project which are iterated upon
Unbounded arrays in Solidity can cause gas-related issues due to their potential for excessive growth, leading to increased computational complexity and resource consumption. Since Ethereum gas fees are determined by the amount of computational effort required to execute a transaction, operations involving large, unbounded arrays can result in prohibitively high costs for users. Additionally, if a function iterates over an unbounded array, it may exceed the gas limit set by the network, causing the transaction to fail. To avoid these issues, developers should use bounded arrays or alternative data structures like mappings, ensuring efficient resource management and a better user experience.

*Instances (2)*:
```solidity
File: example/DelegateRegistry.sol

339:         outgoingDelegationHashes[from].push(delegationHash);

340:         incomingDelegationHashes[to].push(delegationHash);

```

### [L-10] Unsafe ERC20 operation(s)

*Instances (4)*:
```solidity
File: example/CreateOfferer.sol

121:             if (!IERC20(erc20Order.info.tokenContract).approve(address(delegateToken), erc20Order.amount)) {

```

```solidity
File: example/DelegateToken.sol

369:             IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);

393:             IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);

```

```solidity
File: example/DelegateTokenTransferHelpers.sol

41:         IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);

```
