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