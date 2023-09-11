# Low-Findings

## [L-01] Use 0.8.19 Solidity Version because later versions have PUSH0 included that does not work on other evm chains except eth mainnet.

Although PUSH0 opcode is now included with the latest solidity compiler, you need to be careful while using it on any other chain than ETH mainnet.
There are still other chains like L2s that don’t recognize the PUSH0 opcode. Therefore, if you try to use the latest compiler to deploy your contract on any such chain that doesn't support this opcode.

**_8 Instances - 8 Files_**

```solidity
File : src/libraries/RegistryHashes.sol

1: // SPDX-License-Identifier: CC0-1.0
2: pragma solidity ^0.8.21;

```
[1-2](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L1C1-L2C25)

```solidity
File : src/libraries/RegistryOps.sol

1: // SPDX-License-Identifier: CC0-1.0
2: pragma solidity ^0.8.21;

```
[1-2](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryOps.sol#L1C1-L2C25)

```solidity
File : src/libraries/RegistryStorage.sol

1: // SPDX-License-Identifier: CC0-1.0
2: pragma solidity ^0.8.21;

```
[1-2](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L1C1-L2C25)

```solidity
File : src/DelegateRegistry.sol

1: // SPDX-License-Identifier: CC0-1.0
2: pragma solidity ^0.8.21;

```
[1-2](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L1C1-L2C25)

```solidity
File : src/CreateOfferer.sol

1: // SPDX-License-Identifier: CC0-1.0
2: pragma solidity ^0.8.21;

```
[1-2](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L1C1-L2C25)

```solidity
File : src/DelegateToken.sol

1: // SPDX-License-Identifier: CC0-1.0
2: pragma solidity ^0.8.21;

```
[1-2](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L1C1-L2C25)

```solidity
File : src/MarketMetadata.sol

1: // SPDX-License-Identifier: CC0-1.0
2: pragma solidity ^0.8.21;

```
[1-2](https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L1C1-L2C25)

```solidity
File : src/PrincipalToken.sol

1: // SPDX-License-Identifier: CC0-1.0
2: pragma solidity ^0.8.21;

```
[1-2](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L1C1-L2C25)

# Non-Critical

## [NC-01] Some Contracts are not following proper solidity style guide layout

/ Layout of Contract: According to solidity Docs
// version
// imports
// errors
// interfaces, libraries, contracts
// Type declarations
// State variables
// Events
// Modifiers
// Functions

// Layout of Functions:
// constructor
// receive function (if exists)
// fallback function (if exists)
// external
// public
// internal
// private
// internal & private view & pure functions
// external & public view & pure functions
https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout

**Note: These instances missed by bot report**

**_4 Instance - 2 Files_**

### Write view and pure functions at the end of contract after all state changeable methods

```solidity
File : src/DelegateToken.sol


65: function supportsInterface(bytes4 interfaceId) external pure returns (bool) {

78: function onERC1155BatchReceived(address, address, uint256[] calldata, uint256[] calldata, bytes calldata) external pure returns (bytes4) {

83: function onERC721Received(address operator, address, uint256, bytes calldata) external view returns (bytes4) {

```
[65](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L65),
[78](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L78),
[83](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L83)

```solidity
File : src/PrincipalToken.sol

27:  function _checkDelegateTokenCaller() internal view {

```
[27](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L27)